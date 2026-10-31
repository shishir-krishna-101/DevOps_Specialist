# 07 — Sorting

Sorting means arranging data according to a defined ordering rule. Sorting is one of the most important DSA topics because it appears directly in searching, grouping, ranking, scheduling, interval processing, and many other algorithms.

> **Learning goal:** Understand why sorting works, how major sorting algorithms differ, how to analyze their time and space requirements, and how to choose an appropriate strategy rather than memorizing implementations.

---

## 1. What Does Sorting Mean?

Given a collection such as:

```text
[7, 2, 9, 1, 5]
```

ascending sorting produces:

```text
[1, 2, 5, 7, 9]
```

Descending sorting reverses the ordering:

```text
[9, 7, 5, 2, 1]
```

The ordering rule does not have to be numeric. We can sort records by a field such as:

```text
service name
priority
timestamp
CPU usage
error count
```

The first design question is therefore:

> **What ordering relation does the problem require?**

---

## 2. Why Sorting Matters

Sorting is useful because ordered data often makes later operations easier.

Examples:

- Binary search requires an appropriate ordered sequence.
- Duplicate values become easier to identify when equal values are adjacent.
- Ranking becomes straightforward after ordering by a score.
- Interval problems often become easier after sorting by start or end position.
- Grouping equal keys can become simpler.
- Scheduling problems may depend on processing items in a particular order.

Sorting is therefore often a preprocessing step rather than the final goal.

---

## 3. Important Sorting Properties

When comparing sorting algorithms, do not look only at Big-O time.

Important properties include:

### Time complexity

How does running time grow as the input grows?

### Space complexity

How much additional memory does the algorithm require?

### Stability

A sorting algorithm is **stable** when equal-key records preserve their original relative order.

Example:

```text
Before:
(A, priority=2)
(B, priority=1)
(C, priority=2)

Stable sort by priority:
(B, 1)
(A, 2)
(C, 2)
```

`A` remains before `C` because their sort keys are equal.

Stability matters when records have multiple fields and sorting is performed in stages.

### In-place behavior

An algorithm is commonly described as in-place when it uses only a small amount of additional storage relative to the input, although exact definitions vary.

Always state the implementation assumptions when making a space-complexity claim.

---

## 4. Comparison-Based Sorting

Many general-purpose sorting algorithms determine order by comparing elements.

Examples include:

- bubble sort
- insertion sort
- selection sort
- merge sort
- quicksort
- heap sort

Their strategies differ even though they all use comparisons to establish ordering.

A useful theoretical fact is that comparison-based sorting has a lower bound of **Ω(n log n)** comparisons in the general worst-case model.

This does not mean every sorting problem requires `n log n` time: algorithms that exploit additional structure, such as restricted integer ranges, can use different techniques.

---

## 5. Bubble Sort

Bubble sort repeatedly compares neighboring elements and swaps them when they are out of order.

Example:

```text
[5, 1, 4, 2]
```

One pass moves a large element toward the end:

```text
[1, 4, 2, 5]
```

Repeated passes eventually produce a sorted sequence.

### Typical properties

| Property | Bubble sort |
|---|---|
| Average time | O(n²) |
| Worst-case time | O(n²) |
| Extra space | O(1) for a standard in-place implementation |
| Stable | Yes, when equal elements are not unnecessarily swapped |

An optimized implementation can stop early when a complete pass performs no swaps.

### Why learn it?

Bubble sort is rarely the right choice for production-scale sorting, but it is useful for learning:

- adjacent comparisons
- swapping
- loop invariants
- early termination
- quadratic behavior

---

## 6. Selection Sort

Selection sort repeatedly finds the smallest remaining element and places it into the next output position.

Example:

```text
[5, 1, 4, 2]
```

Find the smallest element:

```text
1
```

Place it at the first position:

```text
[1, 5, 4, 2]
```

Then repeat for the remaining portion.

### Typical properties

| Property | Selection sort |
|---|---|
| Average time | O(n²) |
| Worst-case time | O(n²) |
| Extra space | O(1) for a standard in-place implementation |
| Stable | Not generally stable |

Selection sort is useful for understanding repeated selection and why reducing swaps does not automatically reduce comparison complexity.

---

## 7. Insertion Sort

Insertion sort builds a sorted prefix one element at a time.

Example:

```text
[5, 2, 4, 1]
```

Start with:

```text
[5]
```

Insert `2`:

```text
[2, 5]
```

Insert `4`:

```text
[2, 4, 5]
```

Insert `1`:

```text
[1, 2, 4, 5]
```

### Typical properties

| Property | Insertion sort |
|---|---|
| Best case | O(n) for an already ordered sequence under the usual implementation |
| Average time | O(n²) |
| Worst-case time | O(n²) |
| Extra space | O(1) for an array-style in-place implementation |
| Stable | Yes, with appropriate insertion logic |

Insertion sort can be useful for small inputs or inputs that are already nearly sorted, depending on the implementation and workload.

### Key mental model

Maintain this invariant:

> The portion before the current element is sorted.

Then insert the current element into its correct position within that prefix.

---

## 8. Merge Sort

Merge sort uses divide and conquer.

The high-level process is:

```text
Divide
  ↓
Split the input into smaller parts
  ↓
Recursively sort the parts
  ↓
Merge the sorted parts
```

For example:

```text
[8, 3, 5, 1]

       split
      /     \
  [8, 3]   [5, 1]
```

After sorting each half:

```text
[3, 8]   [1, 5]
```

Merge them:

```text
[1, 3, 5, 8]
```

### Complexity

For the standard merge-sort strategy:

- Time: **O(n log n)**
- Extra space: commonly **O(n)** for array-based merging
- Stable: **Yes**, when the merge step handles equal keys appropriately

The exact space usage depends on the representation and implementation.

### Why merge sort matters

It demonstrates:

- recursion
- divide and conquer
- merging
- recurrence reasoning
- stable sorting

---

## 9. The Merge Step

The merge operation receives two already sorted sequences.

Example:

```text
left  = [2, 7, 9]
right = [1, 4, 8]
```

Compare the first remaining element from each side:

```text
2 vs 1 → take 1
2 vs 4 → take 2
7 vs 4 → take 4
7 vs 8 → take 7
9 vs 8 → take 8
```

Then append the remaining value:

```text
[1, 2, 4, 7, 8, 9]
```

A linear merge step is one reason merge sort achieves `O(n log n)` time under the standard recurrence.

---

## 10. Quicksort

Quicksort also uses divide and conquer, but its partitioning strategy differs from merge sort.

A typical conceptual process is:

```text
Choose a pivot
      ↓
Partition elements around the pivot
      ↓
Recursively sort the partitions
```

The quality of the partitions strongly affects performance.

If partitions are reasonably balanced, the recursion depth and total work can be efficient.

If partitions are repeatedly very unbalanced, the worst-case running time can become **O(n²)** for the standard quicksort model.

### Important lesson

Do not say simply:

> Quicksort is O(n log n).

A more accurate statement identifies the case or assumptions being discussed.

Typical comparison:

| Case | Quicksort |
|---|---|
| Average time | O(n log n) under common randomized/typical assumptions |
| Worst-case time | O(n²) for the standard comparison model |
| Extra space | Depends on partition implementation and recursion depth |

Implementation details matter significantly.

---

## 11. Pivot Selection

Pivot selection affects partition quality.

Possible strategies include:

- choosing a fixed position
- choosing a random position
- choosing a value using a small sample

A poor strategy can repeatedly create highly unbalanced partitions.

A randomized strategy can reduce the likelihood of consistently poor partitions under appropriate assumptions, but it does not make the worst-case bound disappear.

Do not claim a specific performance guarantee without stating the algorithm and assumptions.

---

## 12. Heap Sort

Heap sort uses a heap data structure to repeatedly select an extreme element.

High-level idea:

```text
Build heap
   ↓
Extract appropriate element
   ↓
Restore heap property
   ↓
Repeat
```

For a standard binary-heap implementation:

- Time: **O(n log n)**
- Additional auxiliary space: commonly **O(1)** for an in-place array implementation
- Stability: **Not stable**

Heap sort is valuable because it connects sorting with the heap data structure studied later in this roadmap.

---

## 13. Counting Sort

Counting sort is different from comparison-based sorting.

Instead of repeatedly comparing values, it counts occurrences of values from a suitable finite range.

For example, if the values are known to be integers from `0` through `5`:

```text
Input:
[2, 5, 2, 1, 0]

Counts:
0 → 1
1 → 1
2 → 2
3 → 0
4 → 0
5 → 1
```

The counts can then be used to reconstruct the sorted output.

### Important constraint

Counting sort is practical only when the value domain is suitable relative to the number of input elements.

A range from `0` to `10` is very different from a range from `0` to `10^12` for a small input.

Therefore, never describe counting sort as universally `O(n)` without including its range-dependent cost and assumptions.

---

## 14. Radix Sort

Radix sort processes keys digit by digit or character position by character position according to a defined radix representation.

A common least-significant-digit approach repeatedly sorts by one digit position while preserving the ordering established by previous passes.

This is why the stability of the per-digit sorting method can matter.

Radix sort can achieve linear-looking bounds in terms of input length and number of digits under appropriate assumptions, but its complexity depends on:

- number of digits/positions
- radix
- cost of the per-position sorting operation
- representation of the keys

Do not remove those assumptions from the complexity statement.

---

## 15. Comparison of Common Algorithms

The following is a conceptual reference for standard versions of these algorithms.

| Algorithm | Best | Average | Worst | Typical extra space | Stable? |
|---|---:|---:|---:|---:|---|
| Bubble sort | O(n)* | O(n²) | O(n²) | O(1) | Yes* |
| Selection sort | O(n²) | O(n²) | O(n²) | O(1) | No |
| Insertion sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge sort | O(n log n) | O(n log n) | O(n log n) | O(n)* | Yes |
| Quicksort | O(n log n)* | O(n log n)* | O(n²) | Implementation-dependent | Usually no |
| Heap sort | O(n log n) | O(n log n) | O(n log n) | O(1)* | No |

`*` These entries depend on the exact implementation and assumptions. Treat the table as a study reference, not a substitute for analyzing the actual implementation.

---

## 16. Stability Example

Suppose deployment records are:

```text
service=A, environment=prod
service=B, environment=dev
service=C, environment=prod
```

If the records were already ordered by service name and then stably sorted by environment, equal-environment records preserve their earlier relative order.

This property can matter when multiple sorting passes are used.

However, many problems can be solved more directly by providing a composite sort key, so stability should be treated as a requirement when the problem actually needs it rather than as an automatic goal.

---

## 17. In-Place vs Extra Storage

Two algorithms can have the same time complexity but very different memory behavior.

For example:

```text
Algorithm A:
O(n log n) time
O(1) auxiliary space

Algorithm B:
O(n log n) time
O(n) auxiliary space
```

Which is better depends on the constraints.

Consider:

- input size
- available memory
- whether the input may be modified
- whether stable ordering is required
- implementation complexity
- operational environment

Never optimize one resource while ignoring the others.

---

## 18. Python Sorting

Python provides built-in sorting operations for common use cases.

Two important interfaces are:

```python
sorted(iterable)
```

and:

```python
list.sort()
```

The Python documentation describes their behavior and guarantees, including the distinction that `sorted()` returns a new list while `list.sort()` sorts a list in place.

Official documentation:

- https://docs.python.org/3/library/functions.html#sorted
- https://docs.python.org/3/library/stdtypes.html#list.sort

For production Python code, prefer the standard sorting facilities when they satisfy the requirement rather than reimplementing a sorting algorithm without a reason.

The algorithmic implementations in this chapter are primarily for learning and interview preparation.

---

## 19. Sorting With a Key

Many real problems are not about sorting raw numbers.

Suppose records contain:

```python
records = [
    {"service": "api", "latency": 120},
    {"service": "worker", "latency": 80},
    {"service": "db", "latency": 200},
]
```

A common requirement is to order records by a field such as latency.

Python's sorting APIs support a `key` function for defining the value used for comparison.

This is an important practical distinction:

> **Sort by the property the problem actually cares about, not necessarily by the entire object.**

See the official Python sorting documentation:

https://docs.python.org/3/howto/sorting.html

---

## 20. Custom Ordering

A custom ordering may require:

- ascending vs descending order
- primary and secondary fields
- special categories
- normalized strings
- timestamps
- priority values

Before implementing a custom ordering, define it precisely.

For example:

```text
Primary: priority ascending
Secondary: timestamp ascending
Tertiary: service name ascending
```

Then verify that the sorting key actually represents those rules.

Avoid writing a complicated comparator when a clear key function expresses the requirement.

---

## 21. Sorting and Binary Search

Sorting often appears immediately before searching.

For example:

```text
Unsorted data
     ↓
Sort
     ↓
Ordered data
     ↓
Binary search
```

But sorting has a cost.

If the data will be searched only once, sorting first may not be worthwhile.

If the same dataset will be searched many times, preprocessing may be useful.

This leads to an important engineering question:

> **What is the total cost across the entire workload, not just the cost of one operation?**

The searching chapter will analyze this trade-off in more detail.

---

## 22. Sorting Intervals

A common algorithmic pattern is to sort intervals before processing them.

Example:

```text
[5, 8]
[1, 3]
[2, 6]
```

Sorting by start position produces:

```text
[1, 3]
[2, 6]
[5, 8]
```

Once ordered, neighboring intervals can often be processed more systematically.

This pattern appears in problems involving:

- overlap detection
- merging ranges
- scheduling
- resource allocation

The important lesson is not the specific interval problem; it is recognizing when sorting creates the order needed for a simpler linear scan.

---

## 23. Sorting and Deduplication

Sorting can also make duplicate detection straightforward.

Example:

```text
[4, 2, 7, 2, 9, 4]
```

After sorting:

```text
[2, 2, 4, 4, 7, 9]
```

Equal values are adjacent, so a single pass can detect duplicates.

However, a hash set can often detect duplicates in expected linear time without sorting.

Therefore, compare approaches based on the actual requirements:

```text
Need sorted output?
Need stable ordering?
Need only duplicate detection?
Need predictable ordering?
Need repeated queries?
```

The best data structure or algorithm depends on the task.

---

## 24. Common Sorting Mistakes

### Mistake 1 — Memorizing Big-O without assumptions

Always identify the algorithm, implementation, and case.

### Mistake 2 — Assuming every sort is stable

Stability is an algorithm/property question, not a universal sorting guarantee.

### Mistake 3 — Ignoring memory

Two `O(n log n)` algorithms may have different auxiliary-space requirements.

### Mistake 4 — Reimplementing production sorting unnecessarily

Use well-tested standard-library sorting when appropriate.

### Mistake 5 — Sorting when you do not need order

A hash-based approach may be more direct for membership or duplicate detection.

### Mistake 6 — Forgetting the cost of sorting

If sorting is preprocessing, include its cost in the total workload.

### Mistake 7 — Using the wrong sort key

Verify the exact field or composite rule that defines the required order.

---

## 25. How to Choose a Sorting Strategy

Use this decision process.

### Question 1 — Do I actually need sorting?

If the task only requires membership or duplicate detection, another data structure may be better.

### Question 2 — Is the input already nearly ordered?

Insertion-style strategies may perform well for some such workloads, depending on the implementation.

### Question 3 — Is stable ordering required?

If yes, choose an algorithm or library operation with the required stability behavior.

### Question 4 — Is memory constrained?

Compare auxiliary-space requirements.

### Question 5 — Can I exploit restricted key structure?

Counting or radix-style techniques may be appropriate when their assumptions fit.

### Question 6 — Is this production code?

Prefer a mature standard-library implementation when it meets the requirement.

### Question 7 — Is this an interview or learning exercise?

Implementing the requested algorithm may be the goal even when a library sort would be preferable in production.

---

## 26. Complexity Analysis Checklist

For any sorting algorithm, answer:

```text
1. What is the best-case time?
2. What is the average-case time, if meaningful?
3. What is the worst-case time?
4. What extra memory is required?
5. Is it stable?
6. Is it in-place under the stated definition?
7. Does it depend on special input properties?
8. Does the implementation change the bound?
9. What happens for already sorted input?
10. What happens for many equal keys?
```

This is much stronger than memorizing a single Big-O value.

---

## 27. Practice Progression

### Level 1 — Understand

Implement from scratch:

- bubble sort
- selection sort
- insertion sort

For each one, manually trace a small input.

### Level 2 — Divide and conquer

Implement:

- merge sort
- merge operation
- quicksort partitioning

Draw the recursive call structure.

### Level 3 — Analyze

For every implementation, write:

```text
Time:
Space:
Stable?
In-place?
Best case:
Worst case:
```

### Level 4 — Apply

Solve problems involving:

- intervals
- rankings
- duplicate detection
- grouping
- custom records
- scheduling

### Level 5 — Compare

Given a problem, explain why one sorting strategy is more appropriate than another.

---

## 28. Mini Project — Incident Ranking Engine

### Objective

Build a small educational tool that reads incident-like records and produces ordered views.

Example conceptual record:

```text
service
severity
timestamp
error_count
```

### Requirements

Support ordering by:

1. severity
2. error count
3. timestamp
4. service name

Then explain:

- the selected sort key
- whether stable ordering matters
- time complexity
- additional memory usage
- why the chosen approach fits the workload

### Engineering lesson

The goal is not to build a production incident-management system. It is a controlled demonstration of sorting decisions on structured operational data.

> **Repository design decision:** This project is an educational exercise created for this repository. It does not represent an official behavior or implementation of any incident-management product.

---

## 29. Interview Questions

### Fundamentals

1. What does a stable sort mean?
2. What is the difference between in-place and out-of-place sorting?
3. Why does merge sort have `O(n log n)` time?
4. Why can quicksort have `O(n²)` worst-case time?
5. Why can insertion sort be useful for nearly sorted input?

### Reasoning

6. When would you choose merge sort over quicksort?
7. When might sorting be unnecessary?
8. How does sorting help interval problems?
9. How can sorting simplify duplicate detection?
10. Why does counting sort depend on the value range?

### Engineering

11. Why should production code generally prefer a mature standard-library sort when it satisfies the requirement?
12. How would you sort records by multiple fields?
13. When does stability matter?
14. What trade-offs would you consider if memory is limited?
15. How would repeated searches change your decision about whether to sort first?

---

## 30. Completion Checklist

Before moving to the next chapter, you should be able to:

- [ ] Explain why sorting is useful.
- [ ] Explain stability.
- [ ] Explain in-place behavior and its assumptions.
- [ ] Implement bubble sort.
- [ ] Implement selection sort.
- [ ] Implement insertion sort.
- [ ] Explain merge sort.
- [ ] Implement a merge step.
- [ ] Explain quicksort partitioning.
- [ ] Explain quicksort worst-case behavior.
- [ ] Explain heap sort at a high level.
- [ ] Explain why counting sort is not a general-purpose comparison sort.
- [ ] Explain the assumptions behind radix-style sorting.
- [ ] Compare time and space complexity.
- [ ] Use Python's standard sorting facilities appropriately.
- [ ] Choose a sorting strategy based on actual requirements.
- [ ] Analyze the total workload rather than one isolated operation.

---

## 31. Sources and Quality Notes

### Authoritative Python sources

- Python built-in `sorted()`: https://docs.python.org/3/library/functions.html#sorted
- Python `list.sort()`: https://docs.python.org/3/library/stdtypes.html#list.sort
- Python Sorting HOW TO: https://docs.python.org/3/howto/sorting.html

### General algorithmic source

- MIT OpenCourseWare — Introduction to Algorithms: https://ocw.mit.edu/courses/6-006-introduction-to-algorithms-fall-2011/

### Quality policy

- Complexity statements are tied to the stated algorithm and assumptions.
- Implementation details can change space usage, stability, and practical behavior.
- Production-library behavior should be verified against the relevant official documentation.
- Educational implementations are included to teach algorithmic reasoning, not to replace mature standard-library implementations.

---

## Next

Continue to:

**[08 — Searching](../08%20Searching/README.md)**

[← Back to Complete DSA](../README.md)
