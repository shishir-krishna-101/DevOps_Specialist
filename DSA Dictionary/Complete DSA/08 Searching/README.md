# 08 — Searching

Searching means locating information that satisfies a condition. It is one of the most common algorithmic tasks: find a value, determine whether something exists, locate a boundary, or identify the first or last position that meets a rule.

> **Learning goal:** Learn to recognize when linear search is enough, when ordered data enables binary search, how to reason about boundaries, and how search strategies change when the data is structured differently.

---

## 1. What Is Searching?

A search algorithm answers a question about a collection or search space.

Examples:

```text
Does this service exist?
Find the host named "api-01".
Find the first error timestamp after T.
Find the smallest value >= target.
Find whether a configuration contains a required key.
```

The first question should be:

> **What information do I already know about the search space?**

That determines which algorithm is appropriate.

---

## 2. Linear Search

Linear search checks elements one by one until the target is found or the collection is exhausted.

For:

```text
[8, 3, 11, 4, 9]
```

searching for `4` examines:

```text
8 → 3 → 11 → 4
```

A simple implementation is:

```python
def linear_search(items, target):
    for index, value in enumerate(items):
        if value == target:
            return index
    return -1
```

### Complexity

For `n` elements:

- Best case: **O(1)**
- Worst case: **O(n)**
- Extra space: **O(1)** for this implementation

Linear search is often the correct choice when the data is unsorted and only a small number of searches is required.

---

## 3. When Linear Search Is a Good Choice

Do not assume that a more sophisticated algorithm is automatically better.

Linear search can be appropriate when:

- the collection is small
- the data is unsorted
- only one or a few searches are required
- preprocessing would cost more than the search saves
- the collection does not provide efficient random access
- the search condition is not compatible with a simpler indexed search

The total workload matters.

For example:

```text
One search:
scan the data

Thousands of searches:
consider whether preprocessing or another data structure is worthwhile
```

---

## 4. Binary Search

Binary search works by repeatedly reducing an ordered search interval.

For an ascending sequence:

```text
[2, 5, 8, 12, 17, 21, 30]
```

To search for `17`, inspect the middle element first.

If the middle value is too small, discard the lower portion.

If it is too large, discard the upper portion.

Each step removes a large part of the remaining search space.

---

## 5. The Key Requirement for Binary Search

Binary search requires more than merely having numbers.

The search space must provide an ordering or monotonic condition that allows one part of the space to be discarded safely.

For a conventional sorted-array search:

```text
values[left ... right]
```

must be ordered according to the comparison being used.

> **Important:** Binary search is not a trick for making arbitrary data faster. Its correctness depends on the structure of the search space.

---

## 6. Binary Search Complexity

Each successful reduction approximately halves the remaining search interval.

The number of iterations therefore grows logarithmically with the input size.

For a conventional array implementation:

- Best case: **O(1)**
- Worst-case time: **O(log n)**
- Iterative auxiliary space: **O(1)**

A recursive implementation also has recursion-depth and call-stack considerations.

---

## 7. Iterative Binary Search

A clear iterative implementation is:

```python
def binary_search(items, target):
    left = 0
    right = len(items) - 1

    while left <= right:
        middle = left + (right - left) // 2

        if items[middle] == target:
            return middle
        if items[middle] < target:
            left = middle + 1
        else:
            right = middle - 1

    return -1
```

The exact midpoint expression is less important than maintaining a correct search invariant.

---

## 8. Binary Search Invariant

A useful invariant is:

> If the target exists, it is still inside the current search interval.

At every iteration:

```text
left ... right
```

represents the portion that has not yet been ruled out.

When the middle value is too small:

```text
left = middle + 1
```

When it is too large:

```text
right = middle - 1
```

The discarded region must be impossible according to the ordering rule.

This is the central correctness argument.

---

## 9. Tracing Binary Search

For:

```text
[2, 5, 8, 12, 17, 21, 30]
```

search for `21`.

Conceptually:

```text
left=0, right=6
middle=3 → 12
21 > 12 → discard left half

left=4, right=6
middle=5 → 21
found
```

Tracing a few small examples is one of the best ways to catch boundary mistakes.

---

## 10. The Most Common Binary-Search Bugs

Binary search is short but easy to implement incorrectly.

Common errors include:

- using the wrong loop condition
- moving `left` or `right` incorrectly
- failing to discard the middle element
- mixing inclusive and exclusive boundaries
- returning the wrong index
- assuming the input is sorted when it is not
- using a comparison that does not match the ordering
- failing on empty input
- failing on a one-element input

Before coding, decide whether your interval is:

```text
[left, right]
```

or:

```text
[left, right)
```

Then keep that convention throughout the implementation.

---

## 11. Exact Match vs Boundary Search

A major improvement in binary-search skill is learning that the target does not have to be a specific value.

You may instead need to find:

- the first position satisfying a condition
- the last position satisfying a condition
- the first value greater than or equal to a target
- the first value greater than a target
- the last value less than or equal to a target

These are often called **boundary searches**.

The same divide-the-search-space idea applies, but the return condition changes.

---

## 12. First Occurrence

Suppose:

```text
[1, 2, 2, 2, 5, 8]
```

and the target is `2`.

A normal binary search may return any matching `2`.

If the problem asks for the **first** occurrence, finding one match is not enough.

After finding a match, continue searching the left side:

```text
found target
    ↓
record candidate
    ↓
search earlier positions
```

This is a reusable pattern.

---

## 13. Last Occurrence

For the same sequence:

```text
[1, 2, 2, 2, 5, 8]
```

finding the last occurrence means continuing toward the right after a match.

Conceptually:

```text
found target
    ↓
record candidate
    ↓
search later positions
```

The key is to define the answer as a boundary, not simply as an arbitrary matching index.

---

## 14. Lower Bound

A common boundary problem is:

> Find the first position whose value is greater than or equal to the target.

For:

```text
[1, 3, 3, 5, 8]
```

and target `3`, the lower-bound position is the first `3`.

For target `4`, it is the position containing `5`.

For a target larger than every value, the result may be the position immediately after the final element, depending on the chosen API convention.

This boundary-search idea is extremely useful in interval, scheduling, and insertion problems.

---

## 15. Upper Bound

Another common boundary is:

> Find the first position whose value is strictly greater than the target.

For:

```text
[1, 3, 3, 5, 8]
```

and target `3`, the upper-bound position is the `5` position.

Lower and upper bounds are often paired to determine how many values fall within a range.

---

## 16. Counting Values in a Range

Suppose a sorted sequence contains:

```text
[1, 2, 2, 3, 5, 5, 5, 8]
```

To count values in `[2, 5]`, find:

```text
first position >= 2
first position > 5
```

The difference between those positions gives the number of elements in the inclusive range under this convention.

This turns repeated scanning into boundary searches.

The method is useful when many range queries are performed on the same sorted data.

---

## 17. Python's `bisect` Module

Python provides the `bisect` module for maintaining insertion points in sorted lists.

Important functions include:

```python
bisect_left(a, x)
bisect_right(a, x)
```

`bisect_left` returns an insertion point before existing equal values, while `bisect_right` returns an insertion point after existing equal values.

Official documentation:

https://docs.python.org/3/library/bisect.html

This is an example of using a standard-library abstraction instead of implementing a boundary search from scratch when the problem matches the API's semantics.

---

## 18. Search Space Does Not Have to Be an Array

Binary-search reasoning can be applied to a numerical answer space when a condition is monotonic.

Imagine a question:

```text
Is candidate X sufficient?
```

Suppose the answer changes only once:

```text
False False False False True True True
```

Then we can search for the transition point.

The values do not have to be stored in an array.

This is commonly called **binary search on the answer**.

---

## 19. Binary Search on the Answer

A generic pattern is:

```text
low = smallest possible answer
high = largest possible answer

while low < high:
    mid = middle candidate

    if mid satisfies the condition:
        high = mid
    else:
        low = mid + 1
```

This pattern is correct only when the predicate has the required monotonic structure.

For example:

```text
candidate size:
1  2  3  4  5  6  7
N  N  N  Y  Y  Y  Y
```

Once `Y` begins, all larger candidates must also satisfy the predicate for this form of binary search.

---

## 20. Recognizing Monotonicity

Before using binary search on an answer, ask:

1. What is the candidate answer?
2. What is the feasibility predicate?
3. If candidate `x` works, do all larger candidates work?
4. Or, if candidate `x` works, do all smaller candidates work?
5. Is the transition guaranteed to occur in one direction?

If the answer is no, binary search may not be justified.

> **Repository design note:** The examples in this chapter are demonstrations of algorithmic reasoning, not claims about any particular production system.

---

## 21. Searching Rotated Sorted Arrays

A rotated sorted array can look like:

```text
[6, 7, 8, 1, 2, 4, 5]
```

The sequence is not globally sorted, but part of it retains sorted structure.

Some search problems exploit that property by determining which half is sorted and then deciding whether the target can lie in that half.

The important lesson is broader than this specific problem:

> **Look for partial structure that lets you eliminate a portion of the search space.**

Do not apply a standard binary search implementation blindly when the input's ordering assumptions have changed.

---

## 22. Searching a 2D Matrix

Two-dimensional search problems can have different structures.

For example, a matrix may be:

- sorted within every row
- sorted within every column
- globally ordered when flattened conceptually
- only partially ordered

The correct algorithm depends on those guarantees.

This is a good reminder that the phrase "sorted matrix" is not precise enough by itself. State exactly what ordering property is guaranteed.

---

## 23. Hash-Based Lookup vs Binary Search

Suppose you need to repeatedly test whether an exact key exists.

Two possible approaches are:

```text
Hash-based structure:
key → value/location

Sorted sequence:
ordered values → binary search
```

A hash table can provide average-case constant-time lookup under the usual hashing assumptions, while binary search on an array provides logarithmic search time.

But the trade-off includes more than lookup complexity:

- memory usage
- ordering requirements
- update cost
- range queries
- iteration order requirements
- preprocessing cost

Choose the data structure based on the workload.

---

## 24. Search vs Preprocessing

Suppose there are `n` records and `q` searches.

A rough comparison might be:

```text
Repeated linear scans:
O(qn)

Sort once + binary searches:
O(n log n + q log n)
```

These are simplified algorithmic models and assume the required operations are available at the stated costs.

The second strategy can be attractive when `q` is large enough to justify preprocessing.

However, if the dataset changes frequently, the cost of keeping it sorted must also be considered.

---

## 25. Search in Linked Lists

Binary search is generally not a natural fit for a linked list because reaching the middle element is not an O(1) indexed operation.

A linked-list search is typically sequential.

This illustrates an important principle:

> **Algorithm complexity depends on both the algorithm and the data structure.**

An algorithm that is excellent for arrays may be inappropriate for a different representation.

---

## 26. Search Trees Preview

Search trees organize values so that comparisons guide traversal toward a likely target.

A binary search tree typically uses the relationship:

```text
left subtree < node < right subtree
```

under the tree's ordering rule.

A search can then follow one path rather than scanning every node.

The next tree chapter will examine the structural assumptions and performance trade-offs in detail.

---

## 27. Searching Graphs

Graphs require a different model because there may be many paths between nodes.

Two foundational graph-search strategies are:

- **Breadth-first search (BFS)** — explores level by level
- **Depth-first search (DFS)** — explores deeply before backtracking

The choice depends on the question being asked.

For example, in an unweighted graph, BFS can be used to find shortest path distance in number of edges from a starting node when the graph representation and traversal conditions are appropriate.

Graph searching will be covered in detail later.

---

## 28. Common Searching Patterns

### Pattern 1 — Exact lookup

Question:

> Does this exact value exist?

Possible tools:

- linear search
- hash-based lookup
- binary search on ordered data

### Pattern 2 — First/last occurrence

Question:

> Where does a repeated value begin or end?

Use boundary-style binary search when ordering permits it.

### Pattern 3 — Range query

Question:

> How many ordered values fall within a range?

Use lower/upper boundaries when the data is sorted.

### Pattern 4 — Feasibility search

Question:

> What is the smallest/largest candidate that satisfies a monotonic predicate?

Consider binary search on the answer.

### Pattern 5 — Structured traversal

Question:

> How do I reach related nodes or states?

Consider tree or graph traversal rather than array-style searching.

---

## 29. Edge Cases to Test

For every search implementation, test at least:

```text
empty input
one element
many elements
first element is target
last element is target
target absent
target smaller than every value
target larger than every value
duplicate values
all values equal
```

For boundary searches, also test:

```text
boundary at the beginning
boundary at the end
no valid position
all positions satisfy the predicate
no positions satisfy the predicate
```

These cases expose most off-by-one errors.

---

## 30. A Repeatable Problem-Solving Process

### Step 1 — Define the search question

Is it exact lookup, a boundary, a count, a feasibility threshold, or a path?

### Step 2 — Inspect the data structure

Ask whether the data is:

- unsorted
- sorted
- hashed
- tree-structured
- graph-structured
- partially ordered

### Step 3 — Identify the exploitable property

Examples:

```text
ordering
hashing
monotonicity
connectivity
```

### Step 4 — Define the invariant

For binary search, state exactly what the current search interval means.

### Step 5 — Test tiny examples

Use one-element, two-element, duplicate, and absent-target cases.

### Step 6 — Analyze the total workload

Include preprocessing, repeated queries, updates, and memory.

### Step 7 — Prefer standard library functionality when appropriate

For Python, use documented standard-library APIs when they match the problem instead of rewriting them unnecessarily.

---

## 31. Practice Progression

### Level 1 — Basic search

1. Linear search
2. Find maximum/minimum
3. Count occurrences
4. Search for a string
5. Find the first matching element

### Level 2 — Binary search

1. Exact target search
2. First occurrence
3. Last occurrence
4. Lower bound
5. Upper bound
6. Search insertion position

### Level 3 — Advanced binary search

1. Search a rotated sorted array
2. Find a peak under defined assumptions
3. Binary search on a monotonic answer
4. Minimize a feasible threshold
5. Maximize a feasible threshold

### Level 4 — Structured search

1. Tree search
2. BFS
3. DFS
4. Search with constraints
5. Multi-source traversal concepts

---

## 32. DevOps-Oriented Mini Project — Configuration Search Toolkit

### Goal

Build a small educational tool that searches a collection of configuration records using multiple strategies.

### Requirements

Support:

- exact key lookup
- linear search over unsorted records
- sorted records with binary-search-style lookup
- first/last matching boundaries where applicable
- comparison of preprocessing cost and repeated-query cost

### Example record

```text
service=api
environment=prod
version=3
```

### Questions to answer

- When is a linear scan simpler?
- When does sorting become worthwhile?
- What assumptions make binary search correct?
- What happens when records are updated?
- Would a dictionary be a better representation for exact key lookup?

> **This is a repository demonstration/experiment, not a claim that production configuration systems use this exact design.**

---

## 33. Interview Questions

### Fundamentals

1. What is the difference between linear and binary search?
2. What condition must hold before binary search can be used on an ordered array?
3. Why is binary search `O(log n)`?
4. What is the worst-case complexity of linear search?
5. Why can binary search be awkward for linked lists?

### Implementation

6. Why can binary-search boundary conditions cause bugs?
7. What invariant does your binary-search loop maintain?
8. How would you find the first occurrence of a duplicate value?
9. How would you find the insertion position for a target?
10. How would you search for the first feasible value in a monotonic answer space?

### Engineering

11. When is sorting before searching worthwhile?
12. How does repeated-query volume affect the choice of algorithm?
13. When would a hash table be preferable to binary search?
14. What changes if the underlying data is frequently updated?
15. Why does the data structure influence the practical usefulness of a search algorithm?

---

## 34. Completion Checklist

You are ready to move on when you can:

- [ ] Explain linear search clearly.
- [ ] Implement linear search without copying a template.
- [ ] Explain the prerequisites for binary search.
- [ ] Implement iterative binary search.
- [ ] State and maintain a binary-search invariant.
- [ ] Handle empty and one-element inputs.
- [ ] Find first and last occurrences.
- [ ] Explain lower and upper bounds.
- [ ] Use Python's `bisect` documentation effectively.
- [ ] Recognize monotonic search spaces.
- [ ] Explain why preprocessing can make repeated queries cheaper.
- [ ] Compare arrays, hashes, trees, and graphs as search structures.
- [ ] Analyze both algorithmic complexity and workload-level cost.

---

## 35. Sources and Quality Notes

### Authoritative Python sources

- Python `bisect`: https://docs.python.org/3/library/bisect.html
- Python `sorted()`: https://docs.python.org/3/library/functions.html#sorted
- Python list sorting: https://docs.python.org/3/library/stdtypes.html#list.sort

### Important distinction

Algorithmic complexity statements in this chapter describe standard algorithmic models and explicitly stated assumptions. Actual performance depends on the implementation, data representation, input distribution, and workload.

When a claim depends on a particular Python API, the official Python documentation is the source of truth.

---

## Navigation

- [← Previous: 07 — Sorting](../07%20Sorting/README.md)
- [Back to Complete DSA](../README.md)
- Next: `09 — Trees and Binary Search Trees`
