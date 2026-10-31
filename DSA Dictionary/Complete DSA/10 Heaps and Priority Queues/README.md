# 10 — Heaps and Priority Queues

Heaps are specialized tree-based structures designed to make it efficient to access the smallest or largest element according to a chosen ordering rule.

A **priority queue** is an abstract data type in which elements are removed according to priority rather than simply according to insertion order.

> **Learning goal:** Understand heap structure, min-heaps, max-heaps, heap operations, priority queues, heap construction, complexity, and practical problem-solving patterns.

---

## 1. Why Heaps Matter

Suppose a system has many pending tasks, but the next task should be selected by priority.

A normal queue gives:

```text
first in → first out
```

A priority queue instead gives:

```text
highest-priority / lowest-priority item → next
```

The exact priority rule is a design decision of the application.

Heaps are useful because they provide an efficient way to maintain access to an extreme element while elements are inserted and removed.

Common algorithmic uses include:

- priority scheduling
- repeatedly selecting the smallest or largest value
- top-k problems
- merging sorted sequences
- shortest-path algorithms such as Dijkstra's algorithm
- event simulation
- maintaining a running median with multiple heaps

These are algorithmic applications, not claims that a particular external DevOps platform internally uses a heap.

---

## 2. Heap vs Priority Queue

These terms are related but not identical.

### Priority queue

A **priority queue** describes the behavior we want:

> Insert items and efficiently retrieve the item with the highest priority.

### Heap

A **heap** is one data structure that can efficiently implement a priority queue.

This distinction is important because an abstract data type describes behavior, while a data structure describes an implementation.

---

## 3. Binary Heap

The most common heap implementation is the **binary heap**.

A binary heap has two important properties.

### Property 1 — Shape property

A binary heap is a **complete binary tree**.

All levels are filled except possibly the last, and the final level is filled from left to right.

Example:

```text
        2
      /   \
     5     8
    / \   /
   9   6 10
```

### Property 2 — Heap-order property

For a **min-heap**:

```text
parent <= children
```

For a **max-heap**:

```text
parent >= children
```

The heap-order property applies between a node and its children. It does **not** mean that the entire tree is sorted.

---

## 4. Min-Heap

In a min-heap, the smallest element is at the root.

Example:

```text
        1
      /   \
     3     5
    / \   /
   7   9 8
```

The root gives direct access to the minimum element.

A min-heap is useful when the next item should be the smallest available value.

Examples:

- earliest event time
- smallest cost
- smallest number
- next shortest tentative distance

---

## 5. Max-Heap

In a max-heap, the largest element is at the root.

Example:

```text
        10
      /    \
     8      9
    / \    /
   3   5  7
```

The root gives direct access to the maximum element.

A max-oriented priority queue is useful when larger priority values should be selected first.

---

## 6. Heap Representation with an Array

A binary heap does not require explicit tree-node objects.

Because the tree is complete, it can be stored compactly in an array-like structure.

Using zero-based indexing, for a node at index `i`:

```text
left child  = 2*i + 1
right child = 2*i + 2
parent      = (i - 1) // 2
```

The parent formula applies when `i > 0`.

Example:

```text
Tree:
        2
      /   \
     5     8
    / \
   9   6

Array:
[2, 5, 8, 9, 6]
```

This representation is one reason binary heaps are practical: the complete-tree shape avoids needing pointers between nodes.

---

## 7. Heap Is Not a Sorted Array

A common beginner mistake is assuming:

```text
heap = sorted structure
```

That is false.

A min-heap guarantees that the root is the minimum and that each parent is no greater than its children.

For example:

```text
        1
      /   \
     4     2
    / \   /
   9   7 5
```

The values are not globally sorted.

The heap invariant is weaker than full sorting, which is why heap operations can be efficient.

---

## 8. Insert into a Heap

To insert an element into a binary heap:

1. Put the new element at the next available position to preserve completeness.
2. Compare it with its parent.
3. If the heap property is violated, swap it with the parent.
4. Continue upward until the heap property is restored.

This upward movement is commonly called **sift up**, **bubble up**, or **heapify up**.

Example for a min-heap:

```text
Before:
        3
      /   \
     5     8

Insert 2:
        3
      /   \
     5     8
    /
   2
```

The `2` violates the relationship with `5`, so swap:

```text
        3
      /   \
     2     8
    /
   5
```

Then compare `2` with `3`:

```text
        2
      /   \
     3     8
    /
   5
```

The invariant is restored.

---

## 9. Insert Complexity

A binary heap has logarithmic height because it is a complete binary tree.

An inserted value may move from a leaf toward the root.

Therefore:

```text
Insert: O(log n)
```

The exact number of swaps depends on the inserted value and the existing heap.

---

## 10. Remove the Root

For a min-heap, removing the root means removing the minimum element.

The usual process is:

1. Save the root value.
2. Move the last element to the root position.
3. Remove the final array position.
4. Compare the new root with its children.
5. Swap with the appropriate child when the heap property is violated.
6. Continue downward until the property is restored.

This downward movement is commonly called **sift down**, **bubble down**, or **heapify down**.

The operation takes:

```text
O(log n)
```

because an element can move down at most the height of the heap.

---

## 11. Peek at the Root

If we only need to inspect the minimum of a min-heap or maximum of a max-heap, no restructuring is required.

The root is at the first array position in the usual zero-based representation.

Therefore:

```text
Peek: O(1)
```

The exact API used to perform the operation depends on the programming language or library.

---

## 12. Python's `heapq`

Python's standard library provides the `heapq` module for heap-based priority queue operations.

Official documentation:

https://docs.python.org/3/library/heapq.html

The module implements a min-heap-oriented interface over a Python list.

Basic example:

```python
import heapq

numbers = []

heapq.heappush(numbers, 5)
heapq.heappush(numbers, 2)
heapq.heappush(numbers, 8)

smallest = heapq.heappop(numbers)
print(smallest)
```

The smallest value is removed first.

> **Verified fact:** `heapq` is part of Python's standard library and provides heap queue algorithms. The exact behavior and API should be checked against the official Python documentation for the Python version being used.

---

## 13. Building a Heap from Existing Data

Sometimes the data already exists in a list.

Instead of inserting every item independently, a heap can be constructed from the existing collection using a heap-building procedure.

Python's `heapq.heapify()` transforms a list into a heap in place.

Official documentation:

https://docs.python.org/3/library/heapq.html#heapq.heapify

The standard heap-building algorithm runs in:

```text
O(n)
```

This is an important result because repeatedly inserting `n` items would generally give an `O(n log n)` total bound, while bottom-up heap construction can build the heap in linear time.

---

## 14. Why Heap Construction Is O(n)

At first, `O(n)` may seem surprising because individual heap operations can take `O(log n)`.

The key is that bottom-up heap construction does not make every node travel all the way from the root to a leaf.

Many nodes are already near the bottom and can move only a small distance.

A small number of nodes can move far, while a large number of nodes can move only a little.

The total work sums to `O(n)`.

This is a useful reminder:

> The complexity of building a data structure is not always the same as repeatedly performing its insertion operation.

---

## 15. Priority Queue Operations

A typical priority queue supports operations conceptually similar to:

| Operation | Purpose | Binary heap complexity |
|---|---|---:|
| Insert | Add an item | `O(log n)` |
| Peek | Inspect highest-priority item | `O(1)` |
| Remove | Remove highest-priority item | `O(log n)` |
| Build heap | Construct from existing items | `O(n)` |

These bounds assume a standard binary heap and the usual operation definitions.

---

## 16. Choosing the Priority

A priority queue is only useful when the priority rule is clear.

For example, suppose tasks have:

```text
priority = 1, 2, 3, ...
```

You must decide whether:

```text
1 = highest priority
```

or:

```text
larger number = higher priority
```

That policy is an application-level design decision.

Do not confuse the numeric representation of priority with the meaning of priority.

---

## 17. Tie-Breaking

Two items may have the same priority.

For example:

```text
Task A → priority 5
Task B → priority 5
```

If deterministic ordering matters, define a tie-breaking rule.

A common conceptual ordering is:

```text
(priority, sequence_number)
```

where the sequence number records insertion order.

This can produce behavior such as:

```text
same priority → earlier insertion first
```

The tie-breaking policy is a repository/application design decision unless an external specification defines it.

---

## 18. Top-K Problems

Heaps are especially useful for **top-k** problems.

Examples:

- find the `k` largest values
- find the `k` smallest values
- maintain the most frequent `k` items
- keep the best `k` candidates while scanning a large dataset

Suppose there are `n` values and we want the `k` largest values.

A common strategy is to maintain a min-heap containing at most `k` candidates.

Conceptually:

```text
for each value:
    add value to heap

    if heap size > k:
        remove smallest
```

The heap then represents the current `k` largest candidates.

Under the standard binary-heap model, this approach is typically:

```text
O(n log k)
```

with `O(k)` additional storage, assuming `k` is bounded independently of the input scan and heap operations dominate the work.

---

## 19. Merge K Sorted Sequences

Suppose several sequences are individually sorted:

```text
A: 1 4 9
B: 2 5 8
C: 3 6 7
```

We want one sorted output.

A heap can store the current smallest candidate from each sequence.

Conceptually:

```text
put first item from every sequence into heap

while heap is not empty:
    remove smallest candidate
    output it
    insert the next item from that same sequence
```

If there are `k` sequences and `n` total elements, the standard heap-based approach has a typical complexity of:

```text
O(n log k)
```

assuming heap operations take `O(log k)` and the heap contains at most one active candidate per sequence.

---

## 20. Heap Sort

A heap can also be used to sort data.

The high-level idea is:

1. Build a heap.
2. Repeatedly remove the root.
3. Place removed elements into the output order.

Heap sort has:

```text
Time: O(n log n)
```

for its standard comparison-based formulation.

Its exact space characteristics depend on the implementation being discussed. In-place heap sort is commonly analyzed with `O(1)` auxiliary space beyond the input array, while a separate output structure requires additional storage.

The important distinction is:

> A heap is a data structure; heap sort is an algorithm that uses heap operations to produce a sorted sequence.

---

## 21. Heap vs Sorting

If you need all elements in sorted order, sorting the complete dataset may be the simplest solution.

If you repeatedly need only the smallest or largest element, a heap may be more appropriate.

If you need only the top `k` items, maintaining a heap of size `k` can avoid sorting every element.

Ask:

```text
Do I need the entire ordering?
        ↓
      sorting

Do I repeatedly need an extreme element?
        ↓
       heap

Do I only need k best candidates?
        ↓
    heap of size k
```

The correct choice depends on the workload rather than on which data structure is more advanced.

---

## 22. Dijkstra's Algorithm Preview

Shortest-path algorithms often need to repeatedly select the vertex with the smallest currently known tentative distance.

A min-priority queue is a natural fit for this repeated selection.

This chapter only introduces the data-structure connection.

Detailed graph algorithms belong in the graph sections of this roadmap.

Do not treat a heap as the shortest-path algorithm itself.

---

## 23. Running Median with Two Heaps

A more advanced pattern maintains two heaps:

```text
max-oriented heap → lower half
min-oriented heap → upper half
```

The two structures are kept approximately balanced.

The middle values can then be obtained efficiently from the roots.

This is a useful example of combining data structures to maintain a changing aggregate.

The exact balancing invariant should be written explicitly before implementation.

---

## 24. Common Heap Pitfalls

### Mistake 1 — Treating a heap as fully sorted

Only the heap invariant is guaranteed.

### Mistake 2 — Confusing min-heap and max-heap behavior

Write down which extreme must be removed first.

### Mistake 3 — Forgetting the shape property

A binary heap must remain complete.

### Mistake 4 — Using the wrong child during sift down

For a min-heap, the smaller child is normally the candidate for the downward swap.

### Mistake 5 — Ignoring ties

If deterministic ordering matters, define tie-breaking explicitly.

### Mistake 6 — Claiming `O(log n)` for every operation

Peeking at the root is `O(1)`, while insertion and root removal are typically `O(log n)`.

### Mistake 7 — Building by repeated insertion without considering alternatives

When all input data is available initially, bottom-up heap construction can be linear time.

---

## 25. Edge Cases

Always test:

- empty heap
- one element
- two elements
- duplicate priorities
- duplicate values
- already ordered input
- reverse-ordered input
- negative values
- very large values
- repeated insertion and removal
- `k = 1` in top-k problems
- `k = n`
- invalid or impossible `k` values when the application has such constraints

For a priority queue, also test equal-priority items and verify that the intended tie-breaking rule is preserved.

---

## 26. A Repeatable Heap Problem-Solving Process

When facing a heap problem:

### Step 1 — Identify the repeated choice

Ask:

> What item do I repeatedly need next?

If the answer is the smallest or largest current candidate, a heap may fit.

### Step 2 — Define the priority

Write the exact ordering rule.

### Step 3 — Decide min vs max orientation

Choose the heap whose root represents the required next candidate.

### Step 4 — Define tie-breaking

If equal priorities matter, define what happens.

### Step 5 — Determine heap size

Ask whether the heap should contain:

```text
all candidates
```

or only:

```text
k candidates
```

### Step 6 — Analyze operations

Count insertions, removals, and heap size.

### Step 7 — Test edge cases

Especially empty input, duplicates, equal priorities, and extreme `k` values.

---

## 27. DevOps-Oriented Mini Project: Priority Task Scheduler

### Goal

Build a small command-line simulation that processes operational tasks according to priority.

Example task model:

```text
Task
├── priority
├── created_at / sequence number
├── name
└── status
```

Possible operations:

```text
add task
peek next task
process next task
show pending tasks
```

### Suggested learning goals

- represent tasks
- define priority ordering
- handle equal priorities
- use a heap-backed priority queue
- measure operation complexity
- test empty and duplicate-priority cases

### Important distinction

This is a **repository demonstration project**.

It is not an implementation of any specific production scheduler or DevOps platform.

---

## 28. Interview Questions

### Fundamentals

1. What is a heap?
2. What is the difference between a heap and a priority queue?
3. What makes a binary heap complete?
4. What is the heap-order property?
5. What is the difference between a min-heap and max-heap?

### Complexity

6. Why is heap insertion `O(log n)`?
7. Why is removing the root `O(log n)`?
8. Why is peeking at the root `O(1)`?
9. Why can bottom-up heap construction be `O(n)`?
10. When would a heap of size `k` be better than sorting all `n` values?

### Problem solving

11. How would you find the `k` largest elements?
12. How would you merge `k` sorted lists?
13. How can a priority queue help shortest-path algorithms?
14. How can two heaps be used to maintain a running median?
15. How would you define deterministic behavior for equal priorities?

### Practical reasoning

16. When would you choose sorting instead of a heap?
17. What happens if a priority queue's ordering rule is ambiguous?
18. Why is a heap not the same thing as a sorted array?

---

## 29. Practice Progression

### Level 1 — Understand

- Draw min-heaps and max-heaps.
- Convert a small tree into its array representation.
- Calculate parent and child indices.
- Trace sift-up and sift-down operations.

### Level 2 — Implement

Implement:

- heap insertion
- root removal
- peek
- heap construction
- priority queue operations

### Level 3 — Apply

Solve:

- kth largest/smallest
- top-k elements
- merge k sorted sequences
- task scheduling
- running median

### Level 4 — Explain

For every solution, explain:

```text
Why a heap?
What is the invariant?
What is the heap size?
How many operations occur?
What is the time complexity?
What is the space complexity?
```

---

## 30. Completion Checklist

Before moving on, you should be able to:

- [ ] Explain a priority queue.
- [ ] Explain the difference between a priority queue and a heap.
- [ ] Explain min-heaps and max-heaps.
- [ ] Explain the complete-tree property.
- [ ] Convert between heap trees and array representations.
- [ ] Trace insertion with sift up.
- [ ] Trace root removal with sift down.
- [ ] Explain heap operation complexity.
- [ ] Explain why bottom-up heap construction is `O(n)`.
- [ ] Use Python's `heapq` according to its official documentation.
- [ ] Recognize top-k problems.
- [ ] Recognize repeated-minimum/repeated-maximum problems.
- [ ] Explain heap-based merging of sorted sequences.
- [ ] Explain the heap connection to shortest-path algorithms.
- [ ] Handle equal priorities explicitly.
- [ ] Build the Priority Task Scheduler mini project.

---

## 31. Sources and Quality Notes

### Authoritative Python source

- Python `heapq` documentation: https://docs.python.org/3/library/heapq.html

### Source policy

Python API behavior in this chapter should be checked against the official Python documentation for the version being used.

Complexity statements describe standard binary-heap algorithms under the assumptions stated in each section. They are algorithmic properties, not performance guarantees for an unspecified production workload.

### Repository design decisions

The following are intentionally chosen for this learning repository:

- examples use small integers so heap behavior can be traced manually;
- priority scheduling examples use explicit priority fields;
- tie-breaking is treated as an explicit design decision;
- the mini project is a teaching simulation rather than a production scheduler.

---

## Next

Continue to:

**[11 — Graphs](../11%20Graphs/README.md)**

[← Back to Complete DSA](../README.md)
