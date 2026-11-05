# 12 — Greedy Algorithms

Greedy algorithms build a solution through a sequence of choices. At each step, the algorithm selects an option that looks best according to a rule defined for the problem.

Greedy algorithms can be efficient and elegant, but there is an important warning:

> A locally best choice does not automatically produce a globally optimal solution.

The central work in a greedy problem is not merely inventing a plausible rule. It is proving that the rule is safe.

---

## Learning objectives

By the end of this chapter, you should be able to:

- Explain what makes an algorithm greedy.
- Recognize the difference between a local choice and a globally optimal solution.
- Identify common greedy patterns involving sorting, intervals, heaps, and graph edges.
- Explain why a greedy rule works using an exchange argument or a related proof.
- Recognize problems where greedy choices fail.
- Analyze the time and space complexity of a greedy implementation.
- Implement common greedy patterns in Python.

## 1. What is a greedy algorithm?

A greedy algorithm constructs a solution incrementally. At each step it chooses one available option according to a fixed decision rule, then continues without revisiting earlier choices.

A typical outline is:

1. Define what a valid solution looks like.
2. Identify the choices available at the current step.
3. Define a rule for selecting the next choice.
4. Check whether the choice keeps the partial solution valid.
5. Repeat until the solution is complete.
6. Prove that the resulting solution is optimal, if optimality is required.

The choice rule depends on the problem. Examples include:

- Select the interval that finishes earliest.
- Select the smallest available edge that does not create a cycle.
- Select the highest-priority task.
- Repeatedly combine the two lowest-frequency items.

These are different algorithms. There is no universal greedy rule that works for every optimization problem.

### Local optimum vs global optimum

- **Local choice:** the best-looking option at the current step according to the algorithm's rule.
- **Global optimum:** the best complete solution among all valid solutions, according to the objective being optimized.

A greedy algorithm is correct only when its sequence of local choices can be shown to lead to a globally optimal solution for the problem's assumptions.

## 2. Properties commonly used to justify greedy algorithms

Two ideas frequently appear in greedy proofs.

### Greedy-choice property

A problem has a greedy-choice property when there is an optimal solution that begins with the choice selected by the greedy rule. After making that choice, the remaining problem can be solved without losing the possibility of an optimal overall solution.

This is a property to establish for a particular problem—not an assumption to make because a rule seems intuitive.

### Optimal substructure

A problem has optimal substructure when an optimal solution can be composed using optimal solutions to appropriately defined smaller subproblems.

Optimal substructure is useful in both greedy algorithms and dynamic programming. By itself, it does not prove that a greedy choice is safe.

## 3. How to prove a greedy algorithm

A convincing explanation should prove why the decision rule cannot make the final answer worse.

### Exchange argument

An exchange argument starts with an optimal solution and shows that one or more of its choices can be exchanged for the greedy choice without making the solution worse.

A common proof structure:

1. Take an optimal solution.
2. Compare its first relevant decision with the greedy decision.
3. Show that replacing the optimal solution's decision with the greedy decision preserves validity.
4. Show that the objective value does not become worse.
5. Reduce the remaining work to a smaller instance of the same problem.

If this exchange can be repeated safely, the greedy choices can be part of an optimal solution.

### Staying-ahead argument

Show that after every step, the greedy solution is at least as good as a competing solution according to a carefully defined measure.

For interval scheduling, for example, choosing the earliest finishing compatible interval leaves at least as much remaining time as choosing an interval that finishes later.

### Cut/property argument

Some graph algorithms use a structural property of cuts to justify selecting a minimum-weight edge crossing a cut. This kind of proof is specific to the graph problem and its assumptions.

**Important:** A proof must match the exact problem definition, constraints, and objective. A proof for one variant does not automatically prove a similar-looking variant.

## 4. Sorting as a greedy enabler

Many greedy algorithms first sort their input so that the next decision can be made efficiently.

Common sorting keys include:

- End time, for interval selection.
- Weight, for minimum spanning tree edge selection.
- Frequency, for repeated minimum combinations.
- Deadline or profit, for particular scheduling formulations.

Sorting does not make an algorithm greedy by itself. The greedy part is the decision rule applied after or during the ordered scan.

Python sorting references:

- [Built-in `sorted()`](https://docs.python.org/3/library/functions.html#sorted)
- [List sorting](https://docs.python.org/3/library/stdtypes.html#list.sort)

## 5. Interval scheduling: select the maximum number of compatible intervals

Suppose each activity has a start time and finish time. Select as many activities as possible, subject to the rule that selected activities do not overlap.

For the standard unweighted activity-selection formulation, a classic greedy rule is:

> Repeatedly select the compatible activity that finishes earliest.

Why this rule is useful: finishing earlier leaves the largest possible remaining time window for subsequent activities.

### Example

Consider activities represented as `(start, finish)`:

```python
activities = [
    (1, 4),
    (3, 5),
    (0, 6),
    (5, 7),
    (8, 9),
    (5, 9),
]
```

Sort by finish time, then scan from left to right. Select an activity if its start time is compatible with the finish time of the previously selected activity.

```python
def select_activities(activities):
    """Return a maximum-size compatible set for the standard
    unweighted activity-selection problem.

    Each activity is a (start, finish) pair.
    """
    ordered = sorted(activities, key=lambda activity: activity[1])

    selected = []
    last_finish = None

    for start, finish in ordered:
        if last_finish is None or start >= last_finish:
            selected.append((start, finish))
            last_finish = finish

    return selected


activities = [(1, 4), (3, 5), (0, 6), (5, 7), (8, 9), (5, 9)]
print(select_activities(activities))
```

The compatibility test above treats an activity beginning exactly when the previous one finishes as compatible. If a problem defines interval boundaries differently, adapt the comparison.

### Complexity

For (n) activities:

- Sorting: (O(n log n)) time.
- Single scan: (O(n)) time.
- Selected output: up to (O(n)) space.
- Sorting and auxiliary-space details can depend on the language/runtime implementation.

Overall time is (O(n log n)) under the usual comparison-sorting model.

### Do not confuse interval variants

These are different problems and may require different algorithms:

- Select the maximum number of non-overlapping intervals.
- Find the minimum number of rooms/resources needed for all intervals.
- Select intervals with maximum total value.
- Find the maximum number of intervals covered by a point.

The earliest-finish greedy rule above solves the standard unweighted maximum-cardinality selection problem. It does not automatically solve weighted interval scheduling.

## 6. Fractional knapsack vs 0/1 knapsack

A knapsack problem has items with weights and values, plus a capacity limit.

### Fractional knapsack

In fractional knapsack, an item can be divided. A standard greedy rule is to consider items in descending value-per-unit-weight order and take as much as possible while capacity remains.

[
	ext{value density} = rac{	ext{value}}{	ext{weight}}
]

This rule is justified by the ability to take fractions of items.

### 0/1 knapsack

In 0/1 knapsack, each item must be either taken completely or not taken. The fractional-knapsack greedy rule is not generally optimal for this version.

For example, consider capacity (W=10):

| Item | Weight | Value | Value / weight |
|---|---:|---:|---:|
| A | 6 | 12 | 2 |
| B | 5 | 10 | 2 |
| C | 5 | 10 | 2 |

A greedy implementation that takes the first item A cannot fit B or C afterwards, producing value 12. Taking B and C produces value 20. The example demonstrates why a ratio-based greedy rule does not solve arbitrary 0/1 knapsack instances.

**Lesson:** Small changes to the problem definition can invalidate a greedy strategy.

## 7. Coin change: a useful greedy counterexample

A common proposed rule is: repeatedly take the largest coin denomination that does not exceed the remaining amount.

This works for some denomination systems, but not for all systems.

Consider denominations:

[
{1, 3, 4}
]

and amount (6).

- Greedy: (4 + 1 + 1), which uses 3 coins.
- Better solution: (3 + 3), which uses 2 coins.

Therefore, largest-coin-first is not a universally correct coin-change algorithm.

Before using a greedy rule for coin change, establish that it is correct for the denomination system and objective in the problem.

## 8. Huffman coding: repeatedly combine the two smallest frequencies

Huffman coding is a classic greedy construction used to build a prefix-code tree from symbol frequencies.

At each step:

1. Select the two nodes with the smallest frequencies.
2. Combine them into a new parent node whose frequency is their sum.
3. Put the new node back into the candidate collection.
4. Repeat until one tree remains.

A min-heap is a natural data structure for repeatedly retrieving the two smallest frequencies.

Python's `heapq` documentation: [heapq — Heap queue algorithm](https://docs.python.org/3/library/heapq.html).

This section describes the construction idea. A complete encoder/decoder must also define how symbols, tree paths, and bit sequences are represented.

## 9. Greedy algorithms in graph problems

Several important graph algorithms use greedy decisions. Their correctness depends on specific graph properties and assumptions.

### Kruskal's minimum spanning tree algorithm

High-level process:

1. Sort edges by non-decreasing weight.
2. Consider edges in that order.
3. Add an edge if it connects two components that are currently separate.
4. Stop when a spanning tree has been formed, when one exists.

A disjoint-set union (DSU) structure can efficiently track connected components and prevent adding an edge that would create a cycle.

### Prim's minimum spanning tree algorithm

High-level process:

1. Start from a vertex.
2. Track edges that connect the current tree to vertices outside it.
3. Repeatedly choose a minimum-weight eligible edge.
4. Add the newly reached vertex and update eligible edges.

A min-heap is commonly used to select the next minimum-weight candidate efficiently.

### Dijkstra's shortest-path algorithm

Dijkstra's algorithm repeatedly finalizes a currently closest unsettled vertex and relaxes its outgoing edges. Its standard correctness guarantee requires non-negative edge weights.

Do not apply Dijkstra to graphs with negative-weight edges and assume the same guarantee. See the graph chapter for broader shortest-path discussion.

These graph algorithms are introduced here as examples of greedy decision-making. Refer to the graph and advanced-graph chapters for implementation details, invariants, and limitations.

## 10. Greedy scheduling and priority queues

A priority queue can help an algorithm repeatedly select the next item according to a priority rule.

Examples of algorithmic tasks include:

- Process the smallest available task cost first.
- Repeatedly combine the two smallest values.
- Select the next minimum-weight graph edge candidate.
- Maintain the highest-priority eligible event.

A priority queue is a data structure, not a proof of correctness. The fact that a heap efficiently returns a minimum or maximum item does not establish that selecting that item is optimal for the overall problem.

Python's `heapq` module provides a min-heap interface: [official documentation](https://docs.python.org/3/library/heapq.html).

## 11. A practical greedy problem-solving workflow

When a problem appears to invite a greedy solution, work through these questions.

1. **Define the objective.** Are you minimizing cost, maximizing count, maximizing value, or finding a feasible construction?
2. **Write down the constraints.** Are items divisible? Can choices be revisited? Are weights non-negative? Can intervals touch at endpoints?
3. **Identify candidate choices.** What options are available at each step?
4. **Propose a decision rule.** State it precisely enough that another person could implement it.
5. **Check feasibility.** Can the choice make the partial solution invalid?
6. **Try small counterexamples.** Include ties, smallest inputs, largest inputs, and cases where an attractive early choice blocks later choices.
7. **Prove the rule.** Use an exchange argument, staying-ahead argument, or an appropriate structural property.
8. **Implement the rule.** Choose sorting, a heap, a set, or another data structure based on the operations required.
9. **Analyze complexity.** Include sorting, heap operations, scans, and auxiliary storage.
10. **Test against a trusted baseline where practical.** For small inputs, exhaustive search or a dynamic-programming solution can help test a greedy implementation. This is a testing technique, not a substitute for a proof.

## 12. Common greedy patterns

| Pattern | Typical decision rule | Important condition |
|---|---|---|
| Interval selection | Choose earliest finishing compatible interval | Standard unweighted maximum-cardinality formulation |
| Fractional knapsack | Choose highest value density first | Items can be divided |
| Huffman construction | Combine two lowest frequencies | Repeated minimum selection |
| Kruskal MST | Add lightest edge that joins separate components | Minimum spanning tree setting |
| Prim MST | Add cheapest edge leaving current tree | Minimum spanning tree setting |
| Dijkstra | Settle closest unsettled vertex | Non-negative edge weights |
| Coin change | Choose largest usable denomination | Only valid for suitable denomination systems |

Treat the condition column as part of the algorithm. Removing assumptions can change correctness.

## 13. When greedy is the wrong approach

Consider another technique when:

- A locally attractive choice can block a better combination later.
- You cannot establish a safe-choice argument.
- The problem requires comparing many combinations of earlier decisions.
- The same subproblems recur and their results can be reused.
- The problem asks for an optimum over discrete choices, such as general 0/1 knapsack.

Possible alternatives:

- **Dynamic programming:** useful when a problem has suitable subproblem structure and overlapping subproblems.
- **Backtracking:** explores candidate choices and abandons invalid or unpromising partial solutions.
- **Branch and bound:** explores a search space while using bounds to prune candidates.
- **Graph algorithms:** model dependencies or relationships explicitly when the problem is naturally graph-shaped.

Do not choose an algorithmic technique by label alone. Model the problem and verify the assumptions.

## 14. Complexity analysis

Greedy algorithms do not have one fixed complexity. Their cost depends on how candidates are selected and updated.

Common building blocks:

| Operation | Typical complexity |
|---|---:|
| Comparison sort | (O(n log n)) |
| One pass through (n) items | (O(n)) |
| Push to a binary heap | (O(log n)) |
| Pop minimum from a binary heap | (O(log n)) |
| Build a heap from (n) items using heapify | (O(n)) |

These are standard algorithmic bounds for the stated operations; actual implementation details and data structures matter.

When analyzing a greedy solution:

- Count sorting separately from scanning.
- Count every heap insertion and removal.
- State whether the output is included in space complexity.
- State assumptions about the data structure.
- Avoid calling an algorithm (O(n)) if it sorts the input first.

## 15. Common mistakes

- Assuming the locally best choice must be globally optimal.
- Confusing optimal substructure with the greedy-choice property.
- Applying a rule proven for one problem variant to another variant.
- Forgetting constraints such as divisibility or non-negative weights.
- Ignoring tie cases in sorting or scheduling.
- Failing to define whether intervals that touch at an endpoint overlap.
- Using a heap without explaining why the extracted item is safe to choose.
- Providing examples but no correctness argument.
- Reporting only the scan complexity while ignoring sorting or heap operations.

## 16. Practice progression

Work through these tasks in order. For each one, write the decision rule, state the assumptions, test edge cases, and provide a correctness argument.

### Beginner

1. Sort tasks by a chosen field and explain what the ordering achieves.
2. Implement standard unweighted activity selection.
3. Find the minimum number of coins for a small fixed denomination set; compare greedy with exhaustive search.
4. Merge overlapping intervals using sorting and a scan. Recognize this as a sorting-and-scan pattern; explain its invariant rather than assuming every interval problem uses the same greedy rule.

### Intermediate

5. Implement fractional knapsack.
6. Construct a Huffman tree using a min-heap.
7. Implement Kruskal's algorithm with a disjoint-set structure.
8. Compare Prim's and Kruskal's approaches on the same connected weighted graph.

### Challenge

9. Find a counterexample to a proposed greedy rule for a problem of your choice.
10. Compare a greedy result with a dynamic-programming or exhaustive-search baseline on small generated inputs.
11. Write a proof for an interval-selection or minimum-spanning-tree greedy choice.

## 17. Mini project — Resource Allocation Planner

<badge color="info">Repository demonstration</badge>

Build a small command-line program that selects non-overlapping maintenance windows or deployment activities.

### Suggested behavior

- Accept a collection of activities with start and finish times.
- Validate that each activity has a well-formed interval.
- Sort activities by finish time.
- Select a maximum-cardinality compatible set using the standard interval-selection rule.
- Display the selected activities in chronological order.
- Report how many activities were selected.

### Suggested test cases

- Empty input.
- One activity.
- Activities that do not overlap.
- Activities that all overlap.
- Activities with equal finish times.
- An activity that starts exactly when the previous selected activity finishes.
- Invalid intervals where finish precedes start.
- Duplicate activities.

### Engineering extension

Add a separate mode for selecting the maximum total value of compatible activities. Do not reuse the unweighted greedy rule without proof; weighted interval scheduling is a different problem and is commonly approached with dynamic programming.

This project is a repository-designed exercise, not a claim about a particular deployment platform.

## 18. Interview questions

Be able to answer these without relying only on memorized definitions:

1. What is a greedy algorithm?
2. Why does a locally optimal choice not always produce a globally optimal result?
3. What is the greedy-choice property?
4. How is optimal substructure different from the greedy-choice property?
5. What is an exchange argument?
6. Why does earliest-finish-time selection work for standard unweighted activity selection?
7. Why does the fractional-knapsack greedy rule not solve general 0/1 knapsack?
8. Give a denomination system where largest-coin-first fails.
9. Why does Kruskal's algorithm need to avoid adding cycle-forming edges?
10. What role does a heap play in Huffman construction or Prim's algorithm?
11. What edge-weight restriction is required for the standard Dijkstra guarantee?
12. How would you test a proposed greedy rule before attempting a proof?
13. When would dynamic programming be a better fit?
14. How do you include sorting and heap operations in complexity analysis?

## 19. Completion checklist

- [ ] I can explain local choice and global optimum.
- [ ] I understand why greedy correctness must be proved.
- [ ] I can describe an exchange argument.
- [ ] I can implement standard activity selection.
- [ ] I can explain the difference between fractional and 0/1 knapsack.
- [ ] I can demonstrate a counterexample to a greedy coin-change rule.
- [ ] I understand the greedy idea behind Huffman construction.
- [ ] I can explain the greedy choices in Kruskal, Prim, and Dijkstra, including their relevant assumptions.
- [ ] I can identify when a heap or sorting step dominates complexity.
- [ ] I can compare a greedy solution against a small exhaustive or dynamic-programming baseline.
- [ ] I can explain when greedy is not appropriate.

## 20. Sources and quality notes

### Python implementation references

- [Python built-in `sorted()`](https://docs.python.org/3/library/functions.html#sorted)
- [Python list sorting](https://docs.python.org/3/library/stdtypes.html#list.sort)
- [Python `heapq`](https://docs.python.org/3/library/heapq.html)

These references support the Python APIs used in the chapter. The algorithm descriptions and proof obligations are presented as educational material; Python documentation is not being cited as proof of the mathematical correctness of each algorithm.

### Content boundaries

- **Verified language/API references:** The links above document Python sorting and heap operations.
- **Algorithmic concepts:** The chapter explains standard greedy formulations and states the assumptions relevant to each example.
- **Repository design decision:** The practice progression and Resource Allocation Planner are learning exercises designed for this repository.
- **Not implied:** A greedy strategy is not correct merely because it is simple, fast, or uses a priority queue. Correctness must be established for the exact problem.

---

[← Previous: 11 — Graphs](../11%20Graphs/README.md) · [Complete DSA home](../README.md) · [Next: 13 — Divide and Conquer](../13%20Divide%20and%20Conquer/README.md)
