# 02 — Arrays and Strings

Arrays and strings are foundational because many problems can be reduced to **ordered data + controlled traversal**.

The goal of this section is not to memorize tricks. It is to learn how to recognize when a problem can be solved by scanning, indexing, maintaining a range, or reusing information from earlier elements.

---

## What You Will Learn

- array traversal and indexing
- dynamic-array concepts
- strings as ordered sequences
- insertion and deletion trade-offs
- frequency counting
- prefix sums
- two-pointer techniques
- sliding windows
- in-place transformations
- common array/string edge cases
- how to choose between straightforward and optimized approaches

---

## 1. Arrays as Ordered Data

An array stores elements in a sequence where positions can be identified by an index.

A simple mental model is:

```text
index:   0   1   2   3
value:  10  20  30  40
```

The exact operations and their costs depend on the implementation and language. For analysis, always distinguish between:

- accessing an element by position
- scanning all elements
- inserting into the middle
- deleting from the middle
- appending at an end

Do not assume that every operation on every array-like structure has the same complexity.

---

## 2. Basic Traversal

The simplest array algorithm is a single pass:

```python
for value in numbers:
    process(value)
```

If every element is processed once, the traversal work is proportional to the number of elements: `O(n)`.

When you only need one pass, avoid adding extra nested work without a reason.

### Example: Find the Maximum

```python
def find_maximum(numbers):
    if not numbers:
        raise ValueError("numbers must not be empty")

    maximum = numbers[0]

    for number in numbers[1:]:
        if number > maximum:
            maximum = number

    return maximum
```

The important reasoning is:

```text
maximum = largest value seen so far
```

That statement acts as the loop invariant.

Typical complexity:

```text
Time:  O(n)
Space: O(1) auxiliary
```

The exact behavior of the surrounding program still depends on how the input is represented and passed to the function.

---

## 3. Index-Based Processing

Sometimes the position matters as much as the value.

```python
for index in range(len(numbers)):
    print(index, numbers[index])
```

Index-based traversal is useful when you need to:

- compare neighboring elements
- modify elements by position
- maintain multiple positions
- calculate distances between positions
- move pointers through a sequence

Use direct iteration when the index is unnecessary. Use indices when position is part of the problem.

---

## 4. Strings Are Ordered Sequences

Strings can be processed character by character:

```python
for character in text:
    process(character)
```

Common string tasks include:

- counting characters
- finding a character or substring
- comparing sequences
- building transformed text
- checking prefixes or suffixes
- detecting patterns
- maintaining a window over characters

Always verify language-specific string behavior before making assumptions about indexing, slicing, mutability, or memory usage.

For Python-specific behavior, use the official documentation linked at the end of this lesson.

---

## 5. Frequency Counting

Many array and string problems become easier when you count how often values occur.

Conceptually:

```text
input
  ↓
count each value
  ↓
use the counts to answer the question
```

For example, given:

```text
["error", "ok", "error", "warning"]
```

we may want a model like:

```text
error   → 2
ok      → 1
warning → 1
```

In Python, a dictionary is commonly used for this kind of key-value mapping.

A straightforward implementation is:

```python
def count_values(values):
    counts = {}

    for value in values:
        counts[value] = counts.get(value, 0) + 1

    return counts
```

If dictionary operations are treated under their usual average-case analysis assumptions, this gives approximately:

```text
Time:  O(n) average-case
Space: O(k)
```

where `k` is the number of distinct values stored in the result.

State the assumptions whenever average-case hash-table complexity is used.

---

## 6. Prefix Sums

A prefix sum stores cumulative information so that later range queries can reuse earlier work.

For values:

```text
[2, 4, 3, 5]
```

a prefix representation can be:

```text
[0, 2, 6, 9, 14]
```

The extra leading zero makes a range calculation convenient.

For a range from index `left` through `right`, the sum can be obtained from two prefix positions:

```text
prefix[right + 1] - prefix[left]
```

### Why It Helps

Without preprocessing, repeatedly calculating many ranges can require scanning each range.

With prefix sums:

```text
Build prefix data → O(n)
Each range sum    → O(1)
Extra storage     → O(n)
```

The trade-off is preprocessing time and additional memory in exchange for faster repeated queries.

---

## 7. Two Pointers

The two-pointer technique uses two positions to control a scan through a sequence.

Common forms include:

```text
left  →
        ← right
```

or:

```text
slow →
fast  →→
```

### Common Signals

Look for problems involving:

- sorted arrays
- pairs of values
- reversing a sequence
- comparing both ends
- removing duplicates in-place
- detecting relationships between nearby positions

### Example: Reverse In Place

```python
def reverse_in_place(values):
    left = 0
    right = len(values) - 1

    while left < right:
        values[left], values[right] = values[right], values[left]
        left += 1
        right -= 1
```

Each pointer moves toward the other.

Typical complexity:

```text
Time:  O(n)
Space: O(1) auxiliary
```

The key invariant is that every completed swap places the correct element at each processed end position.

---

## 8. Sliding Window

A sliding window maintains a contiguous region of an array or string.

Conceptually:

```text
[ a b c d e f g ]
  └─────┘
   window
```

The window can expand, shrink, or move depending on the problem.

### Fixed-Size Window

If every window contains exactly `k` elements, you can maintain a running value rather than recomputing every window from scratch.

For example:

```text
[1, 2, 3, 4, 5]
 └───┘
   k=3
```

Move the window by removing the outgoing value and adding the incoming value.

### Variable-Size Window

A variable window is useful when the window must satisfy a condition.

General reasoning:

```text
expand right
     ↓
condition satisfied?
     ↓ yes
shrink left while allowed
     ↓
record the best/valid window
```

Sliding-window solutions often achieve linear traversal because each boundary moves forward rather than repeatedly restarting the scan.

Do not assume every window problem is `O(n)` automatically. Analyze how many times each pointer can move and what work happens inside the window updates.

---

## 9. In-Place Operations

An in-place approach modifies the existing structure rather than creating another full-size structure.

Potential advantages:

- lower auxiliary memory
- useful when memory is constrained
- can simplify certain transformations

Potential disadvantages:

- the original input is changed
- reasoning can become harder
- aliasing or shared references can matter
- some problems require preserving the original data

Do not optimize for memory automatically. First determine whether mutation is allowed and whether the memory savings matter.

---

## 10. Neighbor Comparisons

Many array problems depend on relationships between adjacent elements.

Example:

```python
def count_increases(numbers):
    count = 0

    for index in range(1, len(numbers)):
        if numbers[index] > numbers[index - 1]:
            count += 1

    return count
```

The loop starts at index `1` because each element is compared with the previous element.

This pattern appears in:

- detecting changes
- monotonic sequences
- run boundaries
- local peaks or drops
- validating adjacent relationships

Always inspect the smallest valid input before using `index - 1` or `index + 1`.

---

## 11. Common Array/String Patterns

| Pattern | Typical Signal |
|---|---|
| Single pass | Need to inspect each element once |
| Frequency map | Count or compare occurrences |
| Prefix sum | Repeated range aggregation |
| Two pointers | Two positions move through a sequence |
| Sliding window | Contiguous region with changing constraints |
| In-place swap | Transform without full extra storage |
| Running minimum/maximum | Best value seen so far |
| Neighbor comparison | Relationship between adjacent elements |
| Binary search | Ordered/partitionable search space |

Pattern recognition should follow problem analysis. Do not force a technique onto a problem simply because it is familiar.

---

## 12. Edge Cases

For array and string problems, deliberately consider:

- empty sequence
- one element
- two elements
- all elements equal
- duplicate values
- already sorted data
- reverse-sorted data
- negative values, when allowed
- very large values
- repeated characters
- empty string
- one-character string
- target absent
- target at the first position
- target at the last position
- window size larger than the sequence, when relevant

The actual valid cases depend on the problem's input contract.

---

## 13. Common Mistakes

Avoid these mistakes:

- using nested loops when a single-pass approach is possible
- modifying input when the contract requires preservation
- forgetting that an empty sequence has no first element
- accessing neighboring indices without checking boundaries
- assuming hash-table operations have unconditional constant-time guarantees
- rebuilding a window from scratch when incremental updates are possible
- using two pointers without identifying what each pointer represents
- using binary search without a valid ordering or partition property
- reporting complexity without accounting for auxiliary structures
- confusing output memory with auxiliary memory

---

## 14. Practice Progression

### Level 1 — Basic Traversal

Practice:

- find minimum and maximum
- sum values
- count matches
- reverse a sequence
- find the first occurrence
- compare neighboring values

### Level 2 — Frequency and Prefix Techniques

Practice:

- frequency counting
- duplicate detection
- prefix sums
- repeated range queries
- character frequency analysis

### Level 3 — Two Pointers

Practice:

- reverse an array
- compare values from both ends
- pair-sum problems on suitable ordered input
- slow/fast pointer problems
- in-place transformations

### Level 4 — Sliding Window

Practice:

- fixed-size window sums
- maximum/minimum window values
- variable-size constraint windows
- substring problems
- longest/shortest valid range problems

### Level 5 — Interview Variations

For each problem, explain:

1. the brute-force idea
2. the bottleneck
3. the data structure or pattern that removes the bottleneck
4. correctness reasoning
5. time complexity
6. space complexity
7. edge cases

---

## 15. Mini Project — Sequence Analysis Toolkit

Build a small command-line or library-style learning project that accepts a controlled collection of values and provides operations such as:

- count values
- find minimum and maximum
- calculate prefix information
- answer range queries
- detect duplicates
- reverse the sequence
- compute fixed-size window aggregates

> **Demonstration:** The project is intended to exercise the algorithms in this section. It is not presented as a production-ready data-processing system.

A useful project structure is:

```text
input
  ↓
validation
  ↓
analysis operations
  ↓
results
  ↓
tests
```

Keep each operation small enough that its correctness and complexity can be explained independently.

---

## Interview Questions

1. What is the difference between accessing an array element and scanning an array?
2. When would you use a frequency map?
3. What problem does a prefix sum solve?
4. When is a two-pointer approach useful?
5. What is the difference between two pointers and a sliding window?
6. Why can an in-place algorithm use less auxiliary space?
7. What edge cases should you test for an empty sequence?
8. Why should you not assume every hash-table operation is always `O(1)`?
9. How can you optimize repeated fixed-size window calculations?
10. What property must generally hold before binary search is appropriate?
11. How would you compare a brute-force solution with a sliding-window solution?
12. When is modifying the input sequence a bad design choice?

---

## Completion Checklist

- [ ] I can traverse an array in one pass.
- [ ] I understand index-based processing.
- [ ] I can process strings character by character.
- [ ] I can build a frequency map.
- [ ] I understand prefix sums.
- [ ] I can implement a two-pointer scan.
- [ ] I understand fixed-size sliding windows.
- [ ] I understand variable-size sliding windows.
- [ ] I can explain the trade-offs of in-place modification.
- [ ] I can identify array/string edge cases.
- [ ] I can state time and auxiliary-space complexity.
- [ ] I can explain why a chosen pattern fits a problem.

---

## Sources and Quality Notes

For Python-specific behavior, verify implementation details against the official Python documentation:

- Python Tutorial — Data Structures: https://docs.python.org/3/tutorial/datastructures.html
- Python Standard Library — Built-in Types: https://docs.python.org/3/library/stdtypes.html

This lesson focuses on algorithmic reasoning. Complexity statements are models based on stated assumptions rather than universal guarantees for every language or implementation.

---

**[← Back to Complete DSA](../README.md)**
