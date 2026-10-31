# 04 — Linked Lists

Linked lists are a useful way to learn how data structures can represent a sequence without requiring every element to be stored next to the others in memory.

This chapter focuses on the **structure, pointer/reference reasoning, common operations, complexity trade-offs, and problem-solving patterns** behind linked lists.

> **Learning goal:** Learn to reason about nodes and links instead of treating every sequence as an array or Python list.

---

## 1. What Is a Linked List?

A linked list is a sequence of **nodes** where each node stores data and a reference to another node.

A simple singly linked list can be visualized as:

```text
[data | next] → [data | next] → [data | next] → None
```

The final node points to `None`, which represents the end of the list in the usual singly linked-list model.

The important idea is that the sequence is defined by the **links between nodes**, not by numeric indexes into one contiguous sequence abstraction.

---

## 2. Why Study Linked Lists?

Python's built-in `list` is usually the right choice for ordinary sequence work, but linked lists are important because they teach several fundamental ideas:

- references between objects
- explicit node structures
- traversal through links
- insertion and deletion reasoning
- maintaining invariants
- careful pointer/reference updates
- trade-offs between random access and local structural changes

Linked lists also appear frequently in DSA interviews because they test whether you can reason about state changes precisely.

> **Repository design decision:** This chapter teaches linked lists as a DSA concept. It does not suggest replacing Python lists with custom linked lists in normal Python application code.

---

## 3. The Node Model

A singly linked list can be represented conceptually as:

```text
Node
├── value
└── next
```

For example:

```text
head
 ↓
[10 | •] → [20 | •] → [30 | None]
```

`head` identifies the first node.

Each node stores:

1. a value
2. a reference to the next node

The exact class implementation is a learning choice. The important structure is the relationship between nodes.

A minimal Python representation might be:

```python
class Node:
    def __init__(self, value, next_node=None):
        self.value = value
        self.next = next_node
```

This is a **teaching implementation**, not a claim about Python's internal representation of its built-in `list` type.

---

## 4. Head and Tail

Two common references are:

```text
head → first node
tail → last node
```

A simple list may maintain only `head`.

A more complete implementation may also maintain `tail` to make certain end operations easier to express.

Example:

```text
head                         tail
 ↓                             ↓
[10] → [20] → [30] → None
```

The exact fields maintained are an implementation decision.

If an implementation stores a `tail`, its invariants must remain correct after every insertion and deletion.

---

## 5. Traversal

Because linked lists do not provide the same direct indexing model as an array-like sequence, traversal normally follows links from one node to the next.

Conceptually:

```text
current = head

while current is not None:
    process current
    current = current.next
```

For `n` nodes, a complete traversal takes **O(n)** time because each node must be visited.

Auxiliary space is **O(1)** for an iterative traversal when no additional collection proportional to the input is created.

---

## 6. Access by Position

Suppose you want the node at position `k`.

A singly linked list generally requires walking from the head until the desired position is reached.

Conceptually:

```text
head
 ↓
0 → 1 → 2 → 3 → 4
        ↑
        k
```

This means access by position is generally **O(n)** in the worst case.

This is one of the major differences from an array-like structure with direct indexing.

### Important reasoning

Do not confuse:

```text
“I know the index.”
```

with:

```text
“I can reach that element directly.”
```

The data structure determines how the index can be reached.

---

## 7. Searching

To find a value in a singly linked list, you normally traverse the nodes until:

- the value is found, or
- the end is reached.

For `n` nodes:

- Best case: **O(1)** if the first node matches.
- Worst case: **O(n)**.

If the list is unsorted, there is generally no reason to expect binary-search-style logarithmic lookup from the linked structure alone.

---

## 8. Insertion at the Front

Inserting a new node at the beginning can be simple.

Before:

```text
head
 ↓
[A] → [B] → [C]
```

After inserting `X`:

```text
head
 ↓
[X] → [A] → [B] → [C]
```

The essential steps are:

1. create the new node
2. make its `next` reference the old head
3. move `head` to the new node

If the required references are already available, this is **O(1)** time.

---

## 9. Insertion After a Known Node

Suppose you already have a reference to node `B`:

```text
[A] → [B] → [C]
```

To insert `X` after `B`:

```text
[A] → [B] → [X] → [C]
```

The conceptual order matters.

A safe reasoning sequence is:

```text
new.next = current.next
current.next = new
```

The first assignment preserves the rest of the chain before changing the link from `current`.

### Complexity

If the target node is already known, the local insertion can be **O(1)**.

If you first have to search for the target node, the overall operation can become **O(n)**.

This distinction is essential in complexity analysis.

---

## 10. Deletion

Deletion is another operation where link updates matter more than syntax.

Consider:

```text
[A] → [B] → [C]
```

If `B` must be removed, the chain needs to become:

```text
[A] → [C]
```

A common singly linked-list approach is to keep a reference to the node before the target and update its `next` reference.

Conceptually:

```text
previous.next = target.next
```

If the predecessor is already known, the local link change is **O(1)**.

Finding that predecessor may require **O(n)** traversal.

---

## 11. The Most Important Linked-List Skill: Reference Reasoning

Many linked-list bugs come from changing a reference before preserving the part of the chain that is still needed.

Consider:

```text
A → B → C → D
```

If you need to reconnect `A` to `C`, reason about the references first:

```text
A.next = C
```

But if the algorithm needs the old `B` reference later, make sure it has been saved before overwriting the relevant link.

### Practical rule

Before changing a link, ask:

> **What part of the chain could become unreachable after this assignment?**

This question prevents many pointer/reference mistakes.

---

## 12. Reversing a Singly Linked List

Reversing a linked list is a classic interview problem.

Before:

```text
A → B → C → None
```

After:

```text
A ← B ← C
          ↑
         head
```

An iterative solution usually maintains three references conceptually:

```text
previous
current
next_node
```

At each step:

1. save the next node
2. reverse the current node's link
3. move `previous` forward
4. move `current` forward

Pseudocode:

```text
previous = None
current = head

while current is not None:
    next_node = current.next
    current.next = previous
    previous = current
    current = next_node

head = previous
```

### Complexity

For `n` nodes:

- Time: **O(n)**
- Auxiliary space: **O(1)** for the iterative approach

The key lesson is not memorizing the code. It is understanding why `next_node` must be saved before the link is reversed.

---

## 13. Fast and Slow Pointers

A powerful linked-list pattern uses two references that move at different speeds.

Typical names are:

```text
slow
fast
```

A common arrangement is:

```text
slow → one step at a time
fast → two steps at a time
```

This pattern can help solve problems involving:

- finding a middle position
- detecting cycles
- comparing positions relative to the end

The exact initialization and stopping condition depend on the problem.

Do not memorize one version as universal. Define what each pointer means and maintain that invariant throughout the traversal.

---

## 14. Finding the Middle

For many linked-list problems, a slow/fast traversal can locate a middle region without first calculating the list length.

Conceptually:

```text
slow moves 1 step
fast moves 2 steps
```

When `fast` reaches the appropriate stopping condition, `slow` is near the middle.

For an even-length list, there are two middle positions, so the exact result depends on the chosen initialization and stopping rule.

### Interview habit

Always state which middle you want:

```text
first middle
or
second middle
```

Then choose the pointer conditions accordingly.

---

## 15. Cycle Detection

A linked list can contain a cycle instead of eventually reaching `None`.

Example:

```text
A → B → C → D
        ↑     ↓
        └─────┘
```

A standard reasoning pattern is to use slow and fast references.

If the two references eventually meet, that provides evidence of a cycle under the algorithm's stated traversal rules.

The key insight is:

> A faster reference can catch a slower reference when traversal occurs inside a cycle.

### Important distinction

Cycle detection and finding the cycle's entry point are related but different tasks.

Do not claim that detecting a meeting point automatically gives the cycle entry without the additional reasoning required by the algorithm.

---

## 16. Doubly Linked Lists

A doubly linked list stores references in both directions.

Conceptually:

```text
None ← [prev | A | next] ↔ [prev | B | next] ↔ [prev | C | next] → None
```

Each node commonly has:

```text
previous
value
next
```

### Advantages

The structure supports traversal in both directions and can make certain local insertion/deletion operations easier when the relevant node references are already available.

### Cost

Each node requires another link/reference, increasing the structure's bookkeeping and memory requirements compared with a singly linked representation.

The exact implementation and memory footprint depend on the language and representation.

---

## 17. Circular Linked Lists

In a circular linked list, the final node links back into the list rather than pointing to `None`.

Example:

```text
A → B → C
↑       ↓
└───────┘
```

There is no natural `None` endpoint in the cycle.

This changes traversal logic because a loop cannot simply use:

```python
while current is not None:
```

A circular traversal needs a deliberate stopping condition.

> **Common mistake:** Treating a circular list like a null-terminated list can cause an infinite loop.

---

## 18. Linked List vs Python List

The word “list” can be misleading in Python.

A Python `list` is not the same data structure as the linked list taught in this chapter.

### Conceptual comparison

| Operation / property | Linked list | Python `list` |
|---|---|---|
| Follow to position `k` | Sequential traversal | Index-based access is supported |
| Search by value | Usually linear | Usually linear |
| Insert at front | Can be local if head is available | Requires shifting existing elements |
| Insert after known linked node | Local link changes | Requires position-based sequence changes |
| Memory structure | Nodes connected by references | Python sequence implementation |
| Typical use in Python | DSA learning / specialized structures | General-purpose sequence work |

The table describes the conceptual data-structure trade-offs; exact implementation details should be verified from the language documentation when they matter.

Python's official documentation describes `list` as a mutable sequence and documents its sequence operations and behavior.

---

## 19. Complexity Reference

For a conventional singly linked list, the following are useful conceptual expectations:

| Operation | Typical complexity | Important condition |
|---|---:|---|
| Access by position | O(n) | Must traverse links |
| Search | O(n) worst case | May stop early |
| Traverse all nodes | O(n) | Visits each node |
| Insert at head | O(1) | Head reference available |
| Insert after known node | O(1) | Target node already known |
| Delete after known predecessor | O(1) | Predecessor already known |
| Find target node | O(n) | Requires traversal |
| Reverse iteratively | O(n) | Visits each node once |

These are **model-level complexity statements** for the described linked-list representation, not universal guarantees for every implementation.

---

## 20. Common Pitfalls

### Pitfall 1 — Losing the rest of the chain

Overwrite a `next` reference too early and you may lose access to nodes that have not been processed.

### Pitfall 2 — Forgetting the head update

If an operation changes the first node, `head` must be updated when required.

### Pitfall 3 — Incorrect empty-list handling

Always consider:

```text
head = None
```

### Pitfall 4 — Mishandling a one-node list

A single node often exposes incorrect assumptions about both head and tail.

### Pitfall 5 — Confusing known-node cost with search cost

“Insertion is O(1)” may be true only after the target node has already been located.

### Pitfall 6 — Incorrect loop termination

Be especially careful with circular lists and fast/slow pointer algorithms.

### Pitfall 7 — Forgetting tail invariants

If an implementation maintains `tail`, every operation that changes the final node must keep `tail` correct.

### Pitfall 8 — Mixing up first and second middle

Even-length lists require an explicit definition of what “middle” means.

---

## 21. A Repeatable Problem-Solving Process

When solving a linked-list problem, use this process.

### Step 1 — Draw the nodes

Do not start with code immediately.

Write:

```text
A → B → C → None
```

### Step 2 — Identify the references

Ask:

```text
Where is head?
Do I have tail?
Which node is current?
Which node is previous?
Which node is next?
```

### Step 3 — Define the invariant

For example:

> `current` is the next node that still needs to be processed.

### Step 4 — Preserve references before changing links

If changing `current.next`, determine whether the old `current.next` is needed later.

### Step 5 — Trace one small example

Use three or four nodes before testing a larger case.

### Step 6 — Test structural edge cases

At minimum consider:

- empty list
- one node
- two nodes
- insertion at head
- deletion of head
- deletion of tail
- target not found
- duplicate values
- cycle, when relevant

### Step 7 — Analyze the complete operation

Separate:

```text
cost of locating the node
+
cost of changing the links
```

This prevents misleading complexity claims.

---

## 22. Practice Progression

### Level 1 — Structure

1. Define a node.
2. Build a three-node singly linked list.
3. Traverse the list.
4. Count the nodes.
5. Search for a value.

### Level 2 — Modification

6. Insert at the head.
7. Insert after a known node.
8. Delete the head.
9. Delete after a known predecessor.
10. Maintain a tail reference.

### Level 3 — Core Patterns

11. Reverse a linked list.
12. Find the middle node.
13. Detect a cycle.
14. Find the cycle entry after studying the required algorithm.
15. Remove the nth node from the end.

### Level 4 — Variants

16. Implement a doubly linked list.
17. Implement a circular linked list.
18. Merge two sorted linked lists.
19. Check whether a linked list is a palindrome.
20. Rearrange nodes according to a specified rule.

---

## 23. Mini Project — Linked List Resource Queue

Build a small educational data structure that models a sequence of work items.

Example item fields:

```text
resource_id
operation
priority
```

### Required capabilities

- add an item at the front
- append an item
- remove an item by position or identifier
- search for an item
- traverse all items
- reverse the sequence
- report the number of items

### What to document

For every operation, explain:

1. Which node references are required.
2. What links change.
3. Whether a search is required.
4. Time complexity.
5. Additional space.
6. Empty and single-node behavior.

This is a **repository learning project**, not a recommendation to implement production queues with a custom linked list.

---

## 24. Interview Questions

### Fundamentals

1. What is a linked list?
2. What is stored in a singly linked-list node?
3. What is the role of `head`?
4. Why is random access different from array-like indexing?
5. What is the difference between singly and doubly linked lists?

### Complexity

6. What is the complexity of traversing a linked list?
7. Why is access by position generally O(n)?
8. When can insertion be O(1)?
9. Why can an operation that includes searching still be O(n)?
10. What is the auxiliary-space complexity of iterative reversal?

### Pointer/reference reasoning

11. Why must you often save `next` before changing `current.next`?
12. How would you reverse a singly linked list?
13. How do slow and fast pointers work?
14. How can slow and fast pointers detect a cycle?
15. How would you find a middle node?

### Design

16. When would a linked list be a poor choice?
17. Why is Python's built-in `list` not the same as a linked list?
18. What invariants must a structure with both `head` and `tail` maintain?
19. What changes when the list is circular?
20. How would you test a linked-list implementation thoroughly?

---

## 25. Completion Checklist

You are ready to move on when you can:

- [ ] Explain the node-and-link model.
- [ ] Build and traverse a singly linked list.
- [ ] Explain the role of `head` and `tail`.
- [ ] Analyze access and search complexity.
- [ ] Insert at the head.
- [ ] Insert after a known node.
- [ ] Delete a node safely.
- [ ] Reverse a singly linked list iteratively.
- [ ] Explain why references must be preserved before links are overwritten.
- [ ] Use the slow/fast pointer pattern.
- [ ] Explain cycle detection conceptually.
- [ ] Distinguish singly, doubly, and circular linked lists.
- [ ] Compare linked-list trade-offs with Python's built-in list.
- [ ] Analyze locating cost separately from local link-update cost.
- [ ] Test empty, one-node, and small multi-node cases.

---

## 26. Sources and Quality Notes

### Authoritative Python documentation

- Python Tutorial — Data Structures: https://docs.python.org/3/tutorial/datastructures.html
- Python Standard Types — Sequence Types / Lists: https://docs.python.org/3/library/stdtypes.html

### Source policy

- **Verified fact:** Python's official documentation is the source of truth for the behavior of Python's built-in `list` and related sequence types.
- **DSA model:** The linked-list node structures and algorithms in this chapter are educational data-structure models.
- **Repository design decision:** The examples use Python-style classes and pseudocode to make the structures easier to learn.
- **Complexity model:** Complexity statements describe the conventional linked-list representation introduced in this chapter and explicitly state important preconditions.

If a language-specific behavior or implementation detail is not established by the cited documentation, this chapter does not treat it as a verified Python implementation fact.

---

[← Previous: Arrays and Strings](../02%20Arrays%20and%20Strings/README.md) | [Back to Complete DSA](../README.md)
