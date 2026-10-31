# 01 — Foundations

This section builds the reasoning skills needed before studying individual data structures and algorithms in depth.

The objective is simple: **understand how to analyze a problem before trying to optimize its code.**

---

## What You Will Learn

By the end of this section, you should be comfortable with:

- breaking a problem into smaller parts
- identifying inputs, outputs, and constraints
- estimating time complexity
- estimating space complexity
- recognizing common growth rates
- reasoning about correctness
- using invariants to explain an algorithm
- identifying useful edge cases
- comparing two possible approaches
- understanding when recursion is appropriate

---

## 1. Start With the Problem

Before writing code, answer these questions:

1. **What is the input?**
2. **What is the required output?**
3. **What constraints are known?**
4. **What assumptions are safe?**
5. **What edge cases can break a simple solution?**
6. **What would a straightforward solution look like?**
7. **How large can the input become?**

A useful mental model is:

```text
Problem
   ↓
Inputs + Outputs
   ↓
Constraints
   ↓
Simple solution
   ↓
Correctness
   ↓
Complexity
   ↓
Optimization if necessary
```

Do not optimize before you understand the problem and have a correct baseline.

---

## 2. Time Complexity

Time complexity describes how the amount of work performed by an algorithm grows as the input size grows.

It is normally expressed using **Big-O notation**.

Common growth rates include:

| Complexity | Common Intuition |
|---|---|
| `O(1)` | Constant work |
| `O(log n)` | Repeatedly reduce the search space |
| `O(n)` | Process the input once |
| `O(n log n)` | Common in efficient comparison-based sorting |
| `O(n²)` | Compare many pairs / nested processing |
| `O(2ⁿ)` | Explore many combinations |
| `O(n!)` | Explore many permutations |

These are growth models, not execution-time measurements.

### Example: Constant Work

```python
value = numbers[0]
```

If the operation does not depend on the number of elements, its basic work can be modeled as `O(1)`.

### Example: Linear Work

```python
for number in numbers:
    process(number)
```

If `n` elements are processed once, the loop performs work proportional to `n`: `O(n)`.

### Example: Nested Work

```python
for first in numbers:
    for second in numbers:
        compare(first, second)
```

If each of `n` elements is compared with each of `n` elements, the basic work grows proportionally to `n × n`: `O(n²)`.

---

## 3. Space Complexity

Space complexity describes how additional memory requirements grow with input size.

For example:

```python
result = []

for item in numbers:
    result.append(transform(item))
```

If `result` stores one transformed value for every input element, the additional storage grows with `n`: `O(n)`.

When discussing space complexity, distinguish between:

- **input space** — memory occupied by the input itself
- **auxiliary space** — additional memory required by the algorithm
- **output space** — memory required for the returned result

State which interpretation you are using when the distinction matters.

---

## 4. Best, Average, and Worst Case

The same algorithm can behave differently for different inputs.

For example, a search may find the target immediately, after examining many elements, or not find it at all.

When analyzing an algorithm, clarify whether you are describing:

- best-case behavior
- average-case behavior
- worst-case behavior

For interviews and documentation, **worst-case complexity is often the clearest default**, unless the problem specifically asks for another case.

Do not describe an average-case result as a universal guarantee.

---

## 5. Constants and Lower-Order Terms

Big-O focuses on how growth behaves as the input becomes large.

For example:

```text
3n + 10
```

has linear growth, so it is commonly described as:

```text
O(n)
```

Similarly:

```text
n² + n + 50
```

is commonly described as:

```text
O(n²)
```

This does **not** mean constants and implementation details never matter in real systems. It means Big-O intentionally abstracts away those details when describing asymptotic growth.

---

## 6. Input Size Matters

The acceptable algorithm depends on the size and constraints of the problem.

Consider two approaches:

```text
Approach A → O(n²)
Approach B → O(n log n)
```

For a very small input, either may be perfectly reasonable.

For a very large input, the difference in growth can become significant.

Therefore, ask:

> **How large can the input become?**

before deciding that an algorithm is too slow or sufficiently efficient.

Never claim that a particular complexity is acceptable without considering the actual constraints.

---

## 7. Correctness

A fast algorithm that produces incorrect results is not a successful solution.

A practical correctness process is:

```text
Define what should be true
        ↓
Identify the important invariant
        ↓
Check the initial state
        ↓
Check what each operation preserves
        ↓
Check the termination condition
        ↓
Verify the final result
```

### What Is an Invariant?

An **invariant** is a property that remains true throughout the relevant part of an algorithm.

For example, during a search or sorting algorithm, you may be able to state that a particular portion of the data is already known to satisfy a condition.

Learning to state that condition clearly makes complex algorithms easier to reason about.

---

## 8. Edge Cases

Always test cases that challenge your assumptions.

Useful categories include:

- empty input
- one element
- two elements
- duplicate values
- already sorted input
- reverse-sorted input
- all values equal
- very large values
- negative values when allowed
- missing target
- target at the beginning
- target at the end
- invalid input, when validation is part of the problem

Not every problem supports every category. Use the categories that match the actual input contract.

---

## 9. Recursion Fundamentals

Recursion occurs when a function solves a problem by calling itself on a smaller or simpler version of that problem.

A recursive solution needs a clear:

1. **base case** — when recursion stops
2. **recursive case** — how the problem becomes smaller
3. **progress guarantee** — why repeated calls eventually reach the base case

Example:

```python
def countdown(n):
    if n == 0:
        return

    print(n)
    countdown(n - 1)
```

The base case is `n == 0`.

The recursive call moves toward that base case by decreasing `n`.

### Recursion Checklist

Before writing recursive code, ask:

- What is the smallest valid problem?
- What is the base case?
- What state must each call receive?
- Does every recursive call make progress?
- What happens when the input is empty or minimal?
- How much call-stack space is required?

---

## 10. Iteration vs Recursion

Many recursive algorithms can also be expressed iteratively.

Choose based on clarity, constraints, and the problem structure.

### Recursion can be useful when:

- the problem naturally forms a tree of subproblems
- tree traversal is involved
- backtracking is required
- recursive decomposition makes the reasoning clearer

### Iteration can be useful when:

- the problem is naturally sequential
- explicit stack state is manageable
- avoiding recursive call-stack growth is important
- the iterative version is clearer for the intended audience

There is no universal rule that recursion is better or worse. Explain the trade-off for the specific problem.

---

## 11. Trace Before You Optimize

A small manual trace can expose incorrect assumptions quickly.

Suppose the input is:

```text
[4, 7, 2]
```

Do not immediately jump to a large test case.

Instead, write down:

```text
Initial state
    ↓
Operation 1
    ↓
State change
    ↓
Operation 2
    ↓
State change
    ↓
Final result
```

Tracing is especially useful for:

- loops
- recursion
- pointer movement
- stack operations
- queue operations
- graph traversal
- dynamic programming state changes

---

## 12. Baseline Before Optimization

A good engineering habit is to establish a correct baseline first.

Example reasoning:

```text
Can I solve it directly?
        ↓
Does the solution produce the correct result?
        ↓
What is its complexity?
        ↓
Does the constraint require improvement?
        ↓
Can a different data structure reduce the work?
```

Optimization should be driven by the problem's requirements, not by the assumption that the shortest or most clever code is always better.

---

## 13. Complexity Analysis Checklist

After implementing a solution, ask:

### Time

- How many times does each loop execute?
- Are loops nested?
- Does each operation reduce the problem size?
- Is there recursion?
- Are there repeated searches or repeated computations?
- Does a library operation have documented complexity relevant to the analysis?

### Space

- What additional collections are created?
- Does the result grow with input size?
- Is recursion using call-stack space?
- Are temporary copies created?
- Does the algorithm store visited or processed state?

Then summarize the result clearly:

```text
Time:  O(...)
Space: O(...)
```

Include assumptions when the answer depends on them.

---

## 14. Foundation Exercises

Before moving to the next topic, practice these small exercises.

### Exercise 1 — Count Operations

Given a loop, determine how its work grows with `n`.

### Exercise 2 — Find Complexity

For a nested loop, identify whether the loops are independent or dependent on each other.

### Exercise 3 — Trace a Loop

Take a small list and write the value of important variables after every iteration.

### Exercise 4 — Identify Edge Cases

For a simple search function, list at least five input cases that could expose an incorrect assumption.

### Exercise 5 — Write a Recursive Function

Create a small recursive function with an obvious base case and explain why it terminates.

### Exercise 6 — Compare Two Solutions

For the same problem, write a straightforward solution and a more efficient solution. Explain the difference in time and space complexity.

---

## Interview Questions

Use these questions to test understanding rather than memorization.

1. What does Big-O describe?
2. What is the difference between time complexity and space complexity?
3. Why can `O(n²)` become problematic as `n` grows?
4. What is the difference between worst-case and average-case complexity?
5. What is an algorithmic invariant?
6. Why are edge cases important?
7. What are the essential parts of a recursive solution?
8. How do recursion and iteration differ in memory usage?
9. Why should you establish correctness before optimizing?
10. Why is input size important when evaluating an algorithm?

If you cannot explain an answer without relying on a memorized definition, revisit the relevant concept and trace a concrete example.

---

## Completion Checklist

- [ ] I can explain Big-O in my own words.
- [ ] I can distinguish time and space complexity.
- [ ] I can identify common growth rates.
- [ ] I can reason about nested loops.
- [ ] I understand best-, average-, and worst-case analysis.
- [ ] I can identify useful edge cases.
- [ ] I understand the purpose of an invariant.
- [ ] I can write a recursive function with a clear base case.
- [ ] I can explain recursion termination.
- [ ] I can trace an algorithm manually.
- [ ] I can establish a correct baseline before optimizing.
- [ ] I can explain the time and space complexity of a simple implementation.

---

## Sources and Quality Notes

This lesson teaches general algorithmic reasoning rather than prescribing behavior for a specific production system.

For Python-specific behavior, use the official Python documentation:

- Python Tutorial — Data Structures: https://docs.python.org/3/tutorial/datastructures.html
- Python Standard Library — Built-in Types: https://docs.python.org/3/library/stdtypes.html

Where a future lesson depends on a specific language, library, platform, or implementation detail, verify that behavior against its authoritative documentation before treating it as a repository fact.

---

**[← Back to Complete DSA](../README.md)**
