# 05 — Stacks and Queues

Stacks and queues are fundamental structures for controlling **the order in which work is processed**.

This chapter focuses on the ideas behind LIFO and FIFO processing, their Python representations, common algorithms, complexity reasoning, and practical problem-solving patterns.

> **Learning goal:** Learn to recognize whether a problem is fundamentally about the **most recently added item** or the **earliest waiting item**.

---

## 1. The Core Idea

Two questions identify many stack and queue problems:

```text
Stack → Which item should be processed first?
        The most recently added item.

Queue → Which item should be processed first?
        The earliest waiting item.
```

These are ordering rules rather than merely container types.

---

## 2. Stack — LIFO

A stack follows **Last In, First Out (LIFO)**.

Think of a stack of plates:

```text
        ┌─────┐  ← top
        │  C  │
        ├─────┤
        │  B  │
        ├─────┤
        │  A  │
        └─────┘
```

If `C` was added last, it is removed first.

### Main operations

Conceptually:

```text
push → add an item
pop  → remove the most recent item
peek → inspect the top item without removing it
```

The exact operation names depend on the implementation or library.

---

## 3. Stack Using Python `list`

Python's built-in `list` supports operations that make it suitable for a simple stack representation.

Example:

```python
stack = []

stack.append("A")
stack.append("B")
stack.append("C")

item = stack.pop()
```

After the three additions, the next `pop()` removes `C`.

This is a **repository implementation choice** for teaching the stack abstraction. It is not a claim that every stack must be implemented with a Python list.

Python's official documentation describes `list.append()` and `list.pop()` and discusses using lists as stacks.

---

## 4. Queue — FIFO

A queue follows **First In, First Out (FIFO)**.

Think of a line of people waiting for service:

```text
front                         back
  ↓                            ↓
[A] → [B] → [C] → [D]
```

`A` arrived first, so it is served first.

### Main operations

Conceptually:

```text
enqueue → add to the back

dequeue → remove from the front

peek/front → inspect the next item
```

Again, exact method names depend on the implementation.

---

## 5. Queue Using `collections.deque`

Python's standard library provides `collections.deque`, a double-ended queue.

A queue can be modeled by adding at one end and removing from the other:

```python
from collections import deque

queue = deque()

queue.append("A")
queue.append("B")
queue.append("C")

item = queue.popleft()
```

The first item added, `A`, is removed first.

For general Python queue work, `deque` is an important standard-library structure to understand.

Source: Python's official documentation for `collections.deque`:
https://docs.python.org/3/library/collections.html#collections.deque

---

## 6. Why Not Remove From the Front of a Python List?

A common beginner implementation is:

```python
queue = []
queue.append("A")
queue.append("B")

item = queue.pop(0)
```

Although this expresses FIFO behavior, removing from the beginning of a list requires shifting the remaining sequence elements in the list implementation.

For a queue that performs many front removals, this can make the operation inefficient compared with using an appropriate deque-based representation.

> **Problem-solving lesson:** Choosing a data structure is about the operations your workload performs, not merely about whether the structure can represent the data.

---

## 7. Stack Complexity

For a stack represented by a structure with constant-time end insertion/removal under the stated implementation assumptions:

| Operation | Typical complexity |
|---|---:|
| Push | O(1) |
| Pop | O(1) |
| Peek/top | O(1) |
| Search | O(n) |
| Traverse | O(n) |

These are model-level expectations. Always verify the operations of the actual implementation being used.

---

## 8. Queue Complexity

For a queue implemented with a suitable double-ended structure such as the described `deque` pattern:

| Operation | Typical complexity |
|---|---:|
| Enqueue at back | O(1) |
| Dequeue from front | O(1) |
| Inspect front | O(1) |
| Search | O(n) |
| Traverse | O(n) |

The complexity statements describe the relevant deque operations documented by Python; do not generalize them to arbitrary queue implementations.

---

## 9. When Should You Think “Stack”?

Look for language such as:

```text
most recent
last opened
undo
backtrack
nested
reverse order
previous state
matching pairs
```

Typical examples include:

- undo operations
- browser-history-style backtracking models
- expression parsing
- matching delimiters
- depth-first traversal reasoning
- recursive call behavior
- reversing processing order

The important question is:

> **Does the newest unresolved item need to be handled first?**

If yes, a stack is a strong candidate.

---

## 10. Balanced Delimiters

One of the classic stack problems is checking whether delimiters are correctly nested.

Example:

```text
{ [ ( ) ] }
```

The most recently opened delimiter must be the first one closed.

That is exactly LIFO behavior.

### Reasoning pattern

For each character:

1. If it opens a group, push it.
2. If it closes a group, inspect the most recent opening delimiter.
3. Verify that the pair matches.
4. Remove the matched opening delimiter.
5. At the end, verify that no unmatched opening delimiters remain.

A stack naturally represents the unresolved opening delimiters.

### Complexity

For `n` characters:

- Time: **O(n)** for one pass.
- Auxiliary space: up to **O(n)** for unmatched opening delimiters.

---

## 11. Undo and Redo Reasoning

Undo systems often need to remember previous states or actions in reverse order.

A simplified model can use a stack:

```text
actions

A
B
C  ← newest
```

Undo removes `C` first.

A redo mechanism can be modeled with another stack in systems where redo semantics fit that design:

```text
undo stack
redo stack
```

When an action is undone, it can move between the two histories according to the application's rules.

> **Important:** This is a simplified algorithmic model, not a description of how a particular editor implements undo/redo internally.

---

## 12. Queue Applications

Queues are useful when work should generally be processed in arrival order.

Examples include:

- task waiting lines
- breadth-first traversal
- work scheduling models
- event-processing demonstrations
- request-processing simulations

A queue does not automatically imply that a real production system should use a particular in-memory data structure or messaging technology. The correct implementation depends on the system's requirements.

---

## 13. Breadth-First Search and Queues

Breadth-first search (BFS) explores a graph or tree level by level.

A queue helps preserve the order in which discovered nodes should be processed.

Conceptually:

```text
start
  ↓
queue = [start]

process front
add newly discovered nodes to back
repeat
```

For a graph, a visited structure is typically needed to avoid repeatedly processing the same node when the graph contains cycles or multiple paths to the same node.

### Important distinction

The queue controls **processing order**.

The visited structure controls **which nodes have already been discovered or processed**, according to the algorithm's chosen invariant.

Do not treat these as the same responsibility.

---

## 14. Depth-First Search and Stacks

Depth-first search (DFS) follows a path deeply before backtracking.

This can be expressed using:

- recursion, where the call stack provides the nesting structure, or
- an explicit stack.

An iterative model is:

```text
push start

while stack is not empty:
    node = pop()
    process node
    push appropriate neighbors
```

The exact traversal order depends on when neighbors are added and the order in which they are stored.

> **Interview habit:** Never claim a specific DFS output order without specifying the neighbor ordering and implementation details.

---

## 15. Monotonic Stack

A monotonic stack is a problem-solving pattern in which the stack is maintained according to an ordering property.

For example, an algorithm may maintain values in increasing or decreasing order while scanning the input.

The exact invariant depends on the problem.

Common problem families include:

- next greater element
- next smaller element
- previous greater/smaller element
- temperature-style “next qualifying value” problems
- range-boundary reasoning

### The important idea

Do not memorize “monotonic stack = next greater element.”

Instead ask:

> **What elements can no longer become useful once the current value is processed?**

Those elements may be removed from the stack while preserving the problem's invariant.

Many monotonic-stack algorithms achieve **O(n)** total stack-processing time because each input element is pushed and removed a bounded number of times, under the specific algorithm's conditions.

---

## 16. Queue Variants

Several queue-like structures solve different ordering requirements.

### Simple FIFO queue

```text
first in → first out
```

### Deque

Supports operations at both ends.

```text
front ↔ back
```

### Priority queue

The next item is selected according to priority rather than simply arrival order.

This is an important distinction:

```text
Queue      → arrival-based ordering
Priority   → priority-based ordering
```

A priority queue is therefore covered separately in the later **Heaps and Priority Queues** chapter.

---

## 17. Circular Queue Concept

A circular queue treats storage positions as wrapping around from the end back to the beginning.

Conceptually:

```text
0 → 1 → 2 → 3
↑           ↓
└───────────┘
```

The purpose is to reuse available positions rather than treating the end of the underlying storage as an absolute boundary.

When implementing one manually, define clear invariants for:

- front position
- rear/back position
- number of stored elements
- empty state
- full state

Do not rely on ambiguous index arithmetic.

---

## 18. Stack vs Queue

| Property | Stack | Queue |
|---|---|---|
| Ordering | LIFO | FIFO |
| Add | Top/end | Back |
| Remove | Top/end | Front |
| Typical use | Undo, nesting, backtracking | Waiting work, BFS |
| Core question | “What was added most recently?” | “What has waited longest?” |

When an interview problem seems ambiguous, identify the required processing order before choosing the structure.

---

## 19. Common Pitfalls

### Pitfall 1 — Using the wrong end of the structure

A queue requires consistent front/back semantics. A stack requires consistent top semantics.

### Pitfall 2 — Using `list.pop(0)` for a high-volume queue

The representation can produce unnecessary shifting work. Prefer a suitable deque-based design when the workload requires efficient operations at both ends.

### Pitfall 3 — Forgetting empty-state checks

Before removing or inspecting an item, define what happens when the structure is empty.

### Pitfall 4 — Confusing FIFO with priority processing

A priority queue does not simply mean “a faster queue.” It uses a different ordering rule.

### Pitfall 5 — Assuming traversal order without defining it

For graph algorithms, neighbor ordering can affect the exact order in which nodes are processed.

### Pitfall 6 — Losing the algorithmic invariant

For a monotonic stack, the ordering invariant is the heart of the algorithm. If it is not maintained, the result may be wrong.

### Pitfall 7 — Ignoring memory usage

Stacks and queues can hold up to a large portion of the input or traversal frontier. Include that auxiliary storage in the analysis.

---

## 20. A Repeatable Problem-Solving Process

When a problem suggests a stack or queue, use this checklist.

### Step 1 — Identify the required order

Ask:

```text
Newest first?
Oldest first?
Priority first?
Both ends?
```

### Step 2 — Define the invariant

Examples:

```text
Stack contains unresolved opening delimiters.
Queue contains discovered nodes waiting for processing.
Monotonic stack remains ordered according to the problem rule.
```

### Step 3 — Choose the representation

For Python, consider the standard-library structures appropriate to the operation pattern.

### Step 4 — Trace a small example

Write the structure after every important operation.

### Step 5 — Check empty and boundary states

Test:

- empty structure
- one element
- repeated values
- all elements processed
- maximum expected size

### Step 6 — Analyze both time and space

Count:

- pushes/enqueues
- pops/dequeues
- scans
- stored elements

---

## 21. Practice Progression

### Level 1 — Basic Operations

1. Implement a stack with a Python list.
2. Push and pop values.
3. Inspect the top item safely.
4. Implement a queue with `deque`.
5. Enqueue and dequeue values.

### Level 2 — Recognition

6. Reverse a sequence using a stack.
7. Check balanced delimiters.
8. Simulate a waiting line.
9. Implement a simple BFS traversal.
10. Implement iterative DFS.

### Level 3 — Patterns

11. Solve next greater element.
12. Solve a monotonic-stack temperature-style problem.
13. Compare recursive and iterative DFS.
14. Model a circular queue.
15. Explain why `deque` fits a FIFO workload.

### Level 4 — Interview Reasoning

16. Explain LIFO vs FIFO.
17. Explain why `list.pop(0)` is unsuitable for repeated front removals.
18. State the invariant of a delimiter-matching algorithm.
19. Explain how BFS uses a queue.
20. Explain how DFS can use an explicit stack.

---

## 22. Mini Project — Task Processing Simulator

Build a small simulator that models work arriving and being processed.

### Required capabilities

- add tasks to a waiting queue
- process tasks in FIFO order
- support a stack-based undo model for selected actions
- report pending tasks
- handle an empty queue safely
- record processing order

### Suggested model

```text
incoming tasks → queue → processor
                         ↓
                    processing history
```

For the undo portion:

```text
actions → stack
```

### What to document

For each structure, explain:

1. Why it was selected.
2. What ordering rule it provides.
3. What invariant is maintained.
4. Time complexity of the main operations.
5. Additional space required.
6. Edge cases.

This is a **repository learning project**, not a claim that production job systems should be implemented as an in-memory Python queue.

---

## 23. Interview Questions

### Conceptual

1. What is LIFO?
2. What is FIFO?
3. What is the difference between a stack and a queue?
4. When would you choose a deque?
5. Why is a Python list commonly useful for a simple stack?

### Problem-solving

6. How would you check balanced parentheses?
7. How does BFS use a queue?
8. How can DFS be implemented without recursion?
9. What is a monotonic stack?
10. How would you simulate a queue efficiently in Python?

### Complexity

11. Why can `pop(0)` on a Python list be inefficient for repeated queue operations?
12. What is the expected complexity of stack push/pop for the described representation?
13. What is the auxiliary-space complexity of delimiter matching?
14. Why can BFS require substantial additional memory?
15. Why must a complexity claim identify its implementation assumptions?

### Design

16. When is FIFO processing inappropriate?
17. How is a priority queue different from a FIFO queue?
18. What invariant would you maintain in a monotonic-stack solution?
19. How would you handle an empty queue?
20. How would you test a task-processing simulator?

---

## 24. Completion Checklist

You are ready to move on when you can:

- [ ] Explain LIFO and FIFO clearly.
- [ ] Implement a simple stack.
- [ ] Implement a queue with `deque`.
- [ ] Explain why front-removal from a Python list has different performance characteristics from deque-based removal.
- [ ] Recognize stack problems involving nesting or reverse order.
- [ ] Recognize queue problems involving waiting order.
- [ ] Explain how BFS uses a queue.
- [ ] Explain how DFS can use a stack.
- [ ] Solve a balanced-delimiter problem.
- [ ] Explain the basic monotonic-stack pattern.
- [ ] Distinguish FIFO queues from priority queues.
- [ ] Define and maintain algorithmic invariants.
- [ ] Analyze time and auxiliary space.
- [ ] Test empty and boundary cases.

---

## 25. Sources and Quality Notes

### Authoritative Python documentation

- Python Tutorial — Data Structures: https://docs.python.org/3/tutorial/datastructures.html
- Python `collections.deque`: https://docs.python.org/3/library/collections.html#collections.deque
- Python Standard Types — Sequence Types: https://docs.python.org/3/library/stdtypes.html#sequence-types-list-tuple-range

### Source usage

- **Verified fact:** Python's official documentation is the source of truth for the behavior of the Python data structures referenced here.
- **Repository design decision:** The examples use `list` for a simple stack and `collections.deque` for a queue because those representations make the intended operations easy to demonstrate.
- **Algorithmic explanation:** Complexity statements are tied to the described representation and operation assumptions rather than presented as universal properties of every implementation.

If a future implementation depends on a Python behavior not established by these sources, verify it against the relevant official Python documentation before adding it to the repository.

---

## Next

Continue with:

**[06 — Recursion and Backtracking](../06%20Recursion%20and%20Backtracking/README.md)**

[← Back to Complete DSA](../README.md)
