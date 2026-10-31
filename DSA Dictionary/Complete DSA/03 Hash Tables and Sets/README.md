# 03 — Hash Tables and Sets

Hash tables and sets are fundamental tools for fast lookup, membership testing, counting, grouping, and duplicate detection.

This chapter focuses on **how to recognize when a lookup-oriented data structure can simplify a problem**, how to reason about its complexity, and how to implement the patterns clearly in Python.

> **Learning goal:** Move from repeatedly searching through a sequence to building an index that lets you answer useful questions efficiently.

---

## 1. What Problem Do Hash Tables Solve?

Suppose you have a list of hostnames and want to know whether a particular hostname has already appeared.

A direct scan might look conceptually like this:

```text
for each hostname:
    search through the previously seen hostnames
```

That can repeatedly scan existing data.

A hash-table-based structure changes the approach:

```text
seen = {}

for hostname in hostnames:
    check whether hostname is already indexed
    record hostname
```

The important algorithmic idea is:

> **Precompute or maintain a lookup structure so later queries do not require a full scan.**

Python's `dict` and `set` are the primary built-in structures used for these patterns.

---

## 2. Dictionary / Map

A dictionary stores associations between **keys** and **values**.

Conceptually:

```text
key → value
```

Example:

```python
servers = {
    "web-01": "10.0.0.10",
    "web-02": "10.0.0.11",
}
```

You can retrieve the value associated with a key:

```python
ip = servers["web-01"]
```

### Typical uses

Dictionaries are useful for:

- mapping identifiers to objects
- counting occurrences
- grouping records
- indexing data
- caching computed results
- tracking state by identifier
- implementing lookup-based algorithms

### The key idea

When the problem asks:

> “Given this identifier, what information belongs to it?”

a dictionary should be one of the first structures you consider.

---

## 3. Set

A set represents a collection of **distinct elements**.

Example:

```python
seen_hosts = {"web-01", "web-02", "db-01"}
```

Membership testing is central to set usage:

```python
if "web-01" in seen_hosts:
    print("already seen")
```

### Typical uses

Sets are useful for:

- duplicate detection
- membership testing
- removing duplicates when uniqueness is the goal
- comparing groups of values
- tracking items already processed

### Dictionary vs set

Use a **set** when you mainly need to know whether an item exists.

Use a **dictionary** when you need to associate additional information with a key.

```text
Set:
item → membership

Dictionary:
key → associated value
```

---

## 4. Hashing: The Core Concept

A hash table uses a hash-based indexing strategy to organize key/value or membership information.

You do not need to know the implementation details to solve most algorithm problems, but you should understand the algorithmic consequence:

> A suitable hash-based structure can make repeated lookup substantially cheaper than repeatedly scanning a sequence.

For Python dictionaries and sets, operations such as lookup, insertion, and deletion are commonly analyzed as **average-case O(1)** under the usual assumptions about the hash table and its keys.

This is an average-case complexity statement, **not a guarantee that every operation always takes constant time**.

When analyzing an algorithm, state the assumption instead of writing “hash maps are always O(1).”

---

## 5. Frequency Counting

One of the most important dictionary patterns is counting how often each value occurs.

Example input:

```text
["ERROR", "INFO", "ERROR", "WARNING", "INFO", "ERROR"]
```

Desired result:

```text
ERROR   → 3
INFO    → 2
WARNING → 1
```

Conceptually:

```python
counts = {}

for event in events:
    counts[event] = counts.get(event, 0) + 1
```

The general pattern is:

```text
value → number of occurrences
```

### Why it matters

Frequency maps appear in problems involving:

- logs
- characters
- words
- API events
- resource identifiers
- duplicate records
- top-frequency questions

### Complexity

If there are `n` input items and dictionary operations are treated using their usual average-case assumptions, the counting pass is typically **O(n)** time with **O(k)** additional space, where `k` is the number of distinct values stored.

---

## 6. Duplicate Detection

Suppose you need to determine whether an input contains a duplicate.

A lookup structure can record values that have already appeared:

```python
seen = set()

for item in items:
    if item in seen:
        print("duplicate found")
        break
    seen.add(item)
```

The reasoning is more important than the syntax:

1. Start with an empty set.
2. Read one item.
3. Check whether it has already been seen.
4. If yes, a duplicate exists.
5. Otherwise record it.

This avoids repeatedly comparing every new item against the entire collection.

### Complexity

Under the usual average-case hash-table assumptions:

- Time: **O(n)** for one pass.
- Extra space: **O(k)** for the distinct values stored, up to **O(n)**.

---

## 7. Membership Testing

Many problems can be reframed as:

> “Have I seen this before?”

or:

> “Does this value belong to this collection?”

A set is often the natural structure for this requirement.

Examples:

```text
Is this hostname already registered?
Is this error code known?
Has this resource ID already been processed?
Does this dependency name exist?
```

The key design question is:

> **Do I need the associated data, or only membership?**

If only membership matters, a set is usually the clearer representation.

---

## 8. Grouping and Classification

A dictionary can group multiple records under the same key.

Conceptually:

```text
service → list of events
```

For example:

```text
api     → [event1, event2, event5]
worker  → [event3, event4]
```

A common pattern is:

```python
groups = {}

for record in records:
    key = record["service"]
    groups.setdefault(key, []).append(record)
```

The exact implementation can vary. The algorithmic pattern is:

> **Compute a classification key, then append each item to the group associated with that key.**

This pattern is useful for:

- grouping logs by service
- grouping resources by environment
- grouping tasks by priority
- grouping users by role
- organizing records by category

---

## 9. Counting With a Dictionary

Counting problems often become straightforward when you identify the correct key.

Ask:

> **What exactly am I counting?**

Examples:

```text
status code → count
hostname    → count
user        → count
service     → count
word        → count
character   → count
```

The key should represent the thing whose frequency you need.

This small modeling decision is often the most important part of the problem.

---

## 10. Complement Lookup

Some problems become easier when you store information needed to answer a future query.

The classic example is the “two values sum to a target” pattern.

Given:

```text
numbers = [2, 7, 11, 15]
target = 9
```

When examining `7`, the required complement is:

```text
target - 7 = 2
```

If `2` has already been indexed, the pair has been found.

Conceptually:

```text
current value → what value do I need?
```

This is a general lookup technique, not merely a two-sum trick.

### Reasoning pattern

1. Read the current value.
2. Compute the information needed to complete the condition.
3. Look for that information in the lookup structure.
4. If found, use the stored information.
5. Otherwise record the current value.

Under the usual average-case dictionary assumptions, this can reduce a pair-search problem from a quadratic nested scan to a linear expected-time approach.

---

## 11. Set Operations

Sets also support useful mathematical operations.

For sets `A` and `B`:

```text
Union        → elements in A or B
Intersection → elements in both A and B
Difference   → elements in A but not B
```

These operations are useful when comparing collections.

Example reasoning:

```text
A = resources in environment A
B = resources in environment B
```

Then:

```text
A - B → resources only in A
B - A → resources only in B
A & B → resources shared by both
```

The algorithmic value is that a domain problem can sometimes be expressed directly as a set relationship.

---

## 12. Order vs Membership

A frequent design mistake is choosing a structure based only on the input format.

Ask what the algorithm actually needs.

### Need ordered processing?

A list may be appropriate.

### Need fast membership testing?

Consider a set.

### Need key → value association?

Consider a dictionary.

### Need both order and lookup?

You may need more than one structure, depending on the problem.

The correct data structure is determined by the operations your algorithm needs, not simply by the type of the input.

---

## 13. Hashability in Python

Python dictionaries and sets require elements used as keys or set members to satisfy Python's hashability requirements.

For example, this is valid:

```python
servers = {"web-01": "10.0.0.10"}
```

A common beginner mistake is trying to use a mutable container such as a list as a dictionary key or set element.

When Python rejects a value because it cannot be used as a hashable key/member, do not work around the error blindly. First ask:

> **What should uniquely identify this item?**

Sometimes the correct representation is an immutable value such as a tuple, but that choice must match the problem's semantics.

See Python's official documentation on hashable objects before relying on a specific representation.

---

## 14. Common Pitfalls

### Pitfall 1 — Assuming every hash operation is always O(1)

Use the normal average-case analysis and state the assumption when it matters.

### Pitfall 2 — Using a dictionary when only membership is needed

A set can communicate the intent more clearly.

### Pitfall 3 — Accidentally overwriting values

Dictionary keys are unique. Assigning a new value to an existing key replaces its associated value.

### Pitfall 4 — Forgetting what the key represents

A poor key choice can make an otherwise correct algorithm answer the wrong question.

### Pitfall 5 — Ignoring memory usage

A lookup structure trades additional space for faster access. Always account for the stored entries.

### Pitfall 6 — Modifying a collection while iterating over it

Be deliberate about whether a dictionary or set is being changed during iteration. If the operation changes the collection's size, use a safe pattern such as iterating over a separate snapshot when appropriate.

### Pitfall 7 — Removing duplicates when order matters

A set models uniqueness, not arbitrary application-specific ordering requirements. If the original order has meaning, preserve it deliberately.

---

## 15. A Repeatable Problem-Solving Process

When you see a problem that might benefit from hashing, use this checklist:

### Step 1 — Identify the repeated question

Examples:

```text
Have I seen this value?
How many times did this value occur?
What data belongs to this identifier?
Which records share this property?
```

### Step 2 — Choose the lookup structure

```text
Membership → set
Key/value → dictionary
Counting → dictionary
Grouping → dictionary
```

### Step 3 — Define the key

Be explicit about what uniquely identifies the information you need.

### Step 4 — Decide what to store

Store only information needed for future decisions.

### Step 5 — Analyze complexity

Account for:

- input traversal
- lookup operations
- insertion/update operations
- auxiliary storage

### Step 6 — Test edge cases

Consider:

- empty input
- one element
- all values identical
- all values unique
- repeated keys
- missing keys
- unexpected or invalid values, when relevant

---

## 16. Practice Progression

Work through these in order.

### Level 1 — Recognition

1. Check whether a value exists in a collection.
2. Detect whether a sequence contains duplicates.
3. Count occurrences of each value.
4. Find the first repeated value.

### Level 2 — Modeling

5. Group records by a field.
6. Build a key → value index.
7. Compare two collections using set operations.
8. Find values present in one collection but not another.

### Level 3 — Lookup Patterns

9. Solve a complement-pair problem.
10. Track the latest record associated with each identifier.
11. Use a dictionary to store previously computed results.
12. Combine counting and grouping.

### Level 4 — Interview Reasoning

13. Explain why a lookup structure improves a repeated-search problem.
14. State the average-case complexity assumptions.
15. Explain the space/time trade-off.
16. Compare a list, set, and dictionary for the same problem.

---

## 17. Mini Project — Log Event Index

Build a small tool that reads structured log events and creates useful indexes.

### Input concept

Each event contains fields such as:

```text
service
status
host
```

### Required capabilities

- count events by service
- count events by status
- detect duplicate event identifiers
- find all known services
- group events by service
- identify services with a selected status

### Suggested data structures

```text
service → count
status  → count
service → events
seen IDs → set
```

### What to document

For each operation, explain:

1. Why the chosen structure fits.
2. What the key represents.
3. What information is stored.
4. Expected time complexity.
5. Additional space required.
6. Important edge cases.

This is a **repository learning project**, not a claim about how production logging systems must be implemented.

---

## 18. Interview Questions

### Conceptual

1. What problem does a hash table solve?
2. What is the difference between a set and a dictionary?
3. Why can hashing improve repeated lookup?
4. What does average-case O(1) mean for dictionary or set operations?
5. Why should complexity claims about hashing include assumptions?

### Problem-solving

6. How would you detect duplicates in one pass?
7. How would you count the frequency of each log status?
8. How would you group records by service?
9. How would you find whether two collections share values?
10. How does the complement-lookup pattern work?

### Design

11. When would you choose a set instead of a list?
12. When would you choose a dictionary instead of a set?
13. What is the space trade-off of building a lookup index?
14. What happens if multiple records use the same dictionary key?
15. How would you preserve order while still benefiting from lookup-based reasoning?

---

## 19. Completion Checklist

You are ready to move on when you can:

- [ ] Explain what a hash table is at a conceptual level.
- [ ] Explain the difference between a dictionary and a set.
- [ ] Recognize membership problems.
- [ ] Build a frequency map.
- [ ] Detect duplicates using a set.
- [ ] Group records using a dictionary.
- [ ] Explain complement lookup.
- [ ] Use union, intersection, and difference appropriately.
- [ ] Explain average-case hash-table complexity without claiming it is always O(1).
- [ ] Account for additional memory.
- [ ] Explain why a particular key was chosen.
- [ ] Identify important edge cases.
- [ ] Implement a small lookup-based tool without copying a pattern blindly.

---

## 20. Sources and Quality Notes

### Authoritative Python documentation

- Python Tutorial — Data Structures: https://docs.python.org/3/tutorial/datastructures.html
- Python Standard Types — Mapping Types / dictionary and set behavior: https://docs.python.org/3/library/stdtypes.html
- Python Glossary — Hashable: https://docs.python.org/3/glossary.html#term-hashable

### How this chapter uses sources

- **Verified fact:** Python's official documentation is the source of truth for Python dictionary, set, and hashability behavior.
- **Algorithmic guidance:** Complexity discussions use standard hash-table analysis and explicitly qualify average-case assumptions.
- **Repository design decision:** The examples, learning sequence, checklist, and Log Event Index project are created for this repository to make the concepts practical.

If a claim about Python behavior, a specific API, or implementation detail is not established by the cited authoritative documentation, it should not be presented as fact.

---

## Next

Continue to:

**[04 — Linked Lists](../04%20Linked%20Lists/README.md)**

[← Back to Complete DSA](../README.md)
