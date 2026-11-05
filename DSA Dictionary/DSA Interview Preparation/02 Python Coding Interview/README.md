# 02 — Python Coding Interview

Python is the implementation language used throughout this DSA interview track. This guide focuses on writing correct, readable Python under interview conditions—not on memorizing every feature of the language.

You will practice the language features that commonly appear in algorithm implementations, learn when standard-library containers are useful, and avoid mistakes that can make an otherwise sound algorithm incorrect.

> **Scope:** This is a practical interview-oriented subset of Python. It is not a complete Python language reference. Examples use the Python standard library and avoid requiring third-party packages.

---

## 1. What an interviewer needs to understand from your code

A good coding-interview solution should make these things easy to see:

- What the function receives.
- What it returns.
- Which assumptions it relies on.
- How the algorithm processes the input.
- How edge cases are handled.
- Why the approach is correct.
- How its time and auxiliary-space costs grow.

Prefer straightforward code over clever syntax. A short expression is not automatically clearer or faster, and code that is easy to trace is easier to debug while explaining your reasoning.

## 2. Python essentials to be comfortable with

### Variables and basic types

Python variables refer to objects. You do not need to declare a variable's type before assigning a value.

```python
service_name = "api"
retry_count = 3
cpu_percent = 72.5
is_healthy = True
missing_value = None
```

Common built-in types used in DSA exercises:

| Type | Example | Typical use |
|---|---|---|
| `int` | `42` | Counts, indexes, integer values |
| `float` | `72.5` | Decimal measurements |
| `str` | `"api"` | Text and sequences of characters |
| `bool` | `True` | Conditions |
| `NoneType` | `None` | No value / absent result |
| `list` | `[3, 1, 2]` | Ordered, mutable sequence |
| `tuple` | `(3, 1)` | Ordered, immutable sequence |
| `dict` | `{"api": 3}` | Key-to-value mapping |
| `set` | `{3, 1, 2}` | Unique hashable values |

Do not confuse a value with its type. For example, `"3"` is text, while `3` is an integer.

### Conditions

```python
status = "failed"

if status == "failed":
    result = "investigate"
elif status == "degraded":
    result = "check"
else:
    result = "healthy"
```

Useful comparison and logical operators:

| Operator | Meaning |
|---|---|
| `==`, `!=` | Equal, not equal |
| `<`, `<=`, `>`, `>=` | Numeric or supported ordering comparisons |
| `and` | Both conditions must be true |
| `or` | At least one condition must be true |
| `not` | Negates a truth value |
| `in`, `not in` | Membership test |

Use parentheses when they make a compound condition easier to read.

### Loops

```python
statuses = ["healthy", "failed", "healthy"]

for status in statuses:
    print(status)

attempts = 3
while attempts > 0:
    attempts -= 1
```

Use a `for` loop to visit items in an iterable. Use a `while` loop when repetition depends on a condition that changes during execution.

Avoid changing the collection you are iterating over unless you have deliberately designed for that behavior. When filtering a list, building a new list is often easier to reason about.

### Functions and return values

```python
def count_failures(statuses):
    failures = 0

    for status in statuses:
        if status == "failed":
            failures += 1

    return failures


assert count_failures(["healthy", "failed", "failed"]) == 2
assert count_failures([]) == 0
```

Remember:

- A function begins with `def`.
- Parameters receive the values passed by the caller.
- `return` sends a result back to the caller.
- A function that reaches its end without a `return` statement returns `None`.

Use small functions with one clear responsibility. This makes them easier to test and explain.

## 3. Core containers and the operations you should know

### Lists

A list is an ordered, mutable sequence.

```python
jobs = ["build", "test", "deploy"]

first_job = jobs[0]
last_job = jobs[-1]

jobs.append("notify")
jobs[1] = "security-test"

print(jobs)
```

Useful operations:

| Operation | Purpose |
|---|---|
| `items[i]` | Read an item at an index |
| `items[i] = value` | Replace an item |
| `items.append(value)` | Add one item at the end |
| `items.extend(values)` | Add items from an iterable |
| `items.pop()` | Remove and return the last item |
| `items.pop(i)` | Remove and return the item at index `i` |
| `value in items` | Test membership |
| `len(items)` | Get the number of items |
| `items[a:b]` | Create a slice |

Important distinctions:

- `append(x)` adds `x` as one item.
- `extend(iterable)` adds each item from the iterable.
- `list.sort()` sorts the existing list in place and returns `None`.
- `sorted(iterable)` returns a new sorted list.

Do not write `items = items.sort()`; that replaces the list reference with `None`.

### Strings

Strings are immutable sequences of characters.

```python
message = "ERROR service=api"

print(message.lower())
print(message.startswith("ERROR"))
print("service=" in message)
print(message.split())
```

Common methods include:

- `lower()`, `upper()`
- `strip()`
- `split()`
- `join()`
- `startswith()`, `endswith()`
- `replace()`
- `find()`

Methods that produce modified text return a new string; they do not modify the original string in place.

### Dictionaries

A dictionary maps keys to values.

```python
failure_counts = {
    "api": 2,
    "worker": 1,
}

failure_counts["api"] += 1
failure_counts["database"] = 0

api_failures = failure_counts.get("api", 0)
```

Know these operations:

| Operation | Purpose |
|---|---|
| `mapping[key]` | Read a value; raises `KeyError` if the key is absent |
| `mapping.get(key, default)` | Read with a fallback |
| `mapping[key] = value` | Add or replace a value |
| `key in mapping` | Test whether a key exists |
| `mapping.items()` | Iterate over key-value pairs |
| `del mapping[key]` | Delete a key |

A common interview pattern is to use a dictionary for counting or remembering information seen earlier.

```python
def count_items(values):
    counts = {}

    for value in values:
        counts[value] = counts.get(value, 0) + 1

    return counts


assert count_items(["api", "worker", "api"]) == {
    "api": 2,
    "worker": 1,
}
```

Dictionary keys must be hashable. Lists and dictionaries cannot be used as dictionary keys.

### Sets

A set stores unique hashable elements.

```python
hosts = {"node-a", "node-b", "node-a"}

assert len(hosts) == 2
assert "node-a" in hosts

hosts.add("node-c")
hosts.discard("node-x")
```

Use a set when you need uniqueness or repeated membership checks. Do not depend on a set's iteration order for an algorithm's output.

### Tuples

A tuple is an ordered, immutable sequence.

```python
coordinate = (4, 7)
row, column = coordinate
```

Tuples are useful for returning multiple related values or representing a fixed pair such as a graph coordinate. A tuple is hashable only when all its elements are hashable.

## 4. Iteration helpers and comprehensions

### `enumerate()`

Use `enumerate()` when you need both the index and the value.

```python
services = ["api", "worker", "database"]

for index, service in enumerate(services):
    print(index, service)
```

### `range()`

`range(stop)` produces integer values starting at zero and stopping before `stop`.

```python
for index in range(3):
    print(index)  # 0, 1, 2
```

The stop value is excluded. This matters in loops and binary-search boundaries.

### `zip()`

Use `zip()` to iterate over corresponding items from multiple iterables.

```python
hosts = ["node-a", "node-b"]
statuses = ["healthy", "failed"]

for host, status in zip(hosts, statuses):
    print(host, status)
```

By default, iteration stops when the shortest input iterable is exhausted. Do not assume the inputs have equal lengths unless the problem states or your code checks that condition.

### List comprehensions

A list comprehension builds a list from an iterable.

```python
statuses = ["healthy", "failed", "failed"]
failed_statuses = [
    status
    for status in statuses
    if status == "failed"
]

assert failed_statuses == ["failed", "failed"]
```

Use a normal loop when the comprehension becomes difficult to read, has multiple side effects, or needs complex branching.

## 5. Sorting and custom ordering

`sorted()` returns a new list. `list.sort()` modifies a list in place.

```python
durations = [8, 3, 5]

ascending = sorted(durations)
descending = sorted(durations, reverse=True)

assert ascending == [3, 5, 8]
assert descending == [8, 5, 3]
assert durations == [8, 3, 5]
```

Use the `key` parameter to sort records by a selected field.

```python
services = [
    {"name": "api", "failures": 4},
    {"name": "worker", "failures": 1},
    {"name": "database", "failures": 3},
]

ranked = sorted(
    services,
    key=lambda service: service["failures"],
    reverse=True,
)

assert [item["name"] for item in ranked] == [
    "api",
    "database",
    "worker",
]
```

A `key` function returns the value used for comparison. It does not need to rearrange or modify the original records.

If a field may be missing, define the intended missing-value behavior explicitly. Do not assume every record contains every key.

## 6. Useful standard-library tools for DSA

The standard library includes specialized containers and algorithms. Learn the built-in structures first, then use a specialized tool when it makes the solution clearer.

### `collections.Counter`

Use `Counter` for counting hashable values.

```python
from collections import Counter

statuses = ["healthy", "failed", "failed"]
counts = Counter(statuses)

assert counts["failed"] == 2
```

### `collections.defaultdict`

Use `defaultdict` when a missing key should automatically receive a new value from a factory.

```python
from collections import defaultdict

groups = defaultdict(list)

groups["healthy"].append("api")
groups["failed"].append("worker")

assert groups["healthy"] == ["api"]
```

### `collections.deque`

A deque supports appending and popping at either end. It is commonly useful for queue-based BFS.

```python
from collections import deque

queue = deque()
queue.append("job-1")
queue.append("job-2")

first_job = queue.popleft()

assert first_job == "job-1"
```

For a FIFO queue, prefer `deque.popleft()` over repeatedly removing index zero from a list.

### `heapq`

The `heapq` module provides heap operations. Its standard heap is a min-heap: the smallest item is at index zero.

```python
import heapq

priorities = [7, 2, 5]
heapq.heapify(priorities)

smallest = heapq.heappop(priorities)

assert smallest == 2
```

Use a heap when you repeatedly need access to the smallest item or need to maintain a priority queue. A heap is not the same as a fully sorted list.

### `bisect`

The `bisect` module locates insertion points in a sorted sequence.

```python
from bisect import bisect_left

values = [10, 20, 20, 40]
position = bisect_left(values, 20)

assert position == 1
```

The insertion position is not, by itself, proof that the target exists. If you need membership, check that the position is in range and the value at that position equals the target.

The sequence must already be sorted according to the ordering used by the search.

## 7. Exceptions and defensive boundaries

You should recognize common exceptions raised by ordinary operations:

| Exception | Example situation |
|---|---|
| `IndexError` | Accessing a list index that does not exist |
| `KeyError` | Reading a missing dictionary key with bracket lookup |
| `ValueError` | A value is valid in type but unsuitable for the operation, such as a missing value passed to `list.index()` |
| `TypeError` | An operation is applied to an incompatible type |

Handle an exception when the problem contract requires it. Do not wrap every line in a broad `except Exception` block; doing so can hide programming mistakes.

Example:

```python
def find_position(values, target):
    try:
        return values.index(target)
    except ValueError:
        return -1


assert find_position([4, 8, 2], 8) == 1
assert find_position([4, 8, 2], 7) == -1
```

The return value `-1` is a design choice for this example's function contract, not a universal Python convention.

## 8. Write tests as you solve

Sample inputs are not enough. Test the behavior that matters to the problem.

```python
def first_failed_service(services):
    for service in services:
        if service["status"] == "failed":
            return service["name"]

    return None


assert first_failed_service([
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
]) == "worker"

assert first_failed_service([
    {"name": "api", "status": "healthy"},
]) is None

assert first_failed_service([]) is None
```

For a problem you are solving, consider:

- Empty input, if allowed.
- The smallest valid input.
- One element matching the condition.
- No element matching the condition.
- Duplicate values.
- Negative and zero values when numeric.
- Already sorted and reverse-sorted data when ordering matters.
- Repeated values at the boundaries.
- Input mutation: did your function change something it should preserve?

For larger exercises, use Python's `unittest` framework or another test runner available in the environment. The standard-library reference is linked below.

## 9. Python-specific mistakes to watch for

| Mistake | Why it causes trouble | Better habit |
|---|---|---|
| Using `=` instead of `==` in a comparison | Assignment and equality testing are different operations | Read the condition carefully |
| Forgetting the return value | The function may return `None` | Check every intended output path |
| Using `items = items.sort()` | `sort()` changes the list in place and returns `None` | Call `items.sort()` or use `sorted(items)` |
| Indexing without checking bounds | Can raise `IndexError` | Prove the index is valid before access |
| Using `mapping[key]` for an optional key | Can raise `KeyError` | Use `get()` or test membership |
| Assuming `bisect_left` proves membership | It returns an insertion point | Check the returned position and value |
| Using a list as a queue with repeated `pop(0)` | Front removals shift remaining elements | Use `collections.deque` |
| Using a set when order matters | Set iteration order is not a suitable output contract | Preserve order separately or use a list plus a set |
| Mutating a list while iterating over it | Items may be skipped or processing may become confusing | Build a new collection or iterate over a deliberate copy |
| Treating `None` like zero or an empty string | These values have different meanings and truth behavior | Compare explicitly when the contract distinguishes them |
| Using a mutable object as a dictionary key | Lists and dictionaries are unhashable | Choose an immutable, hashable key representation |

### Mutable default argument pitfall

Avoid using a mutable list or dictionary as a function's default value when you intend a fresh object on each call.

Problematic pattern:

```python
def add_service(name, services=[]):
    services.append(name)
    return services
```

Safer pattern:

```python
def add_service(name, services=None):
    if services is None:
        services = []

    services.append(name)
    return services


assert add_service("api") == ["api"]
assert add_service("worker") == ["worker"]
```

The default expression is evaluated when the function is defined, not afresh for every call. Using `None` and creating the list inside the function avoids sharing that mutable default between calls.

## 10. Complexity: analyze the algorithm, not just the syntax

Python's built-in operations have different costs. For interview analysis, identify the operations your algorithm performs and state the assumptions behind your complexity estimate.

Common high-level patterns:

| Code shape | Typical algorithmic analysis |
|---|---|
| One pass over `n` items | (O(n)) |
| Two independent passes over `n` items | (O(n)) |
| Nested loops each visiting up to `n` items | (O(n^2)) |
| Repeatedly halve a search interval | (O(\log n)) |
| Sort `n` values using Python sorting | (O(n \log n)) comparison-based analysis |
| Store up to `n` items in an auxiliary list/map/set | (O(n)) auxiliary space |

Be precise about the difference between:

- **Input space:** memory occupied by the supplied input.
- **Auxiliary space:** additional memory used by the algorithm.
- **Output space:** memory required for the returned result.

Do not claim that every dictionary or set operation is guaranteed (O(1)) in every possible circumstance. State the expected/average-case assumption when using hash-based lookup in a complexity explanation. If a question requires strict worst-case guarantees, explain that separately.

For detailed algorithm analysis, use [Complete DSA — Foundations](../../Complete%20DSA/01%20Foundations/README.md).

## 11. Interview exercises

Try each task without reading a solution first. Write the function contract, implementation, tests, and complexity analysis.

### Exercise 1 — Count failed jobs

Given a list of status strings, return the number equal to `"failed"`.

Example:

```python
assert count_failed(["healthy", "failed", "failed"]) == 2
assert count_failed([]) == 0
```

### Exercise 2 — Find duplicate hostnames

Given a list of hostnames, return the set of hostnames that occur more than once.

Example:

```python
assert duplicate_hosts(["node-a", "node-b", "node-a"]) == {"node-a"}
assert duplicate_hosts([]) == set()
```

### Exercise 3 — Group service names by status

Given records with `name` and `status` fields, return a dictionary mapping each status to the list of service names with that status.

Example:

```python
records = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "database", "status": "healthy"},
]

assert group_services(records) == {
    "healthy": ["api", "database"],
    "failed": ["worker"],
}
```

### Exercise 4 — Find the first value above a threshold

Given a list of numeric measurements and a threshold, return the index of the first value strictly greater than the threshold. Return `-1` if no such value exists.

Example:

```python
assert first_above([40, 65, 81, 55], 70) == 2
assert first_above([40, 65, 55], 70) == -1
```

### Exercise 5 — Keep the highest-priority jobs

Given a list of integer priorities and a positive integer `k`, return the `k` largest values in descending order. Define and test what should happen when `k` is zero, negative, or larger than the input length before implementing.

This exercise is intended to make you compare sorting with heap-based approaches. Do not assume one is always better; explain the trade-off for the stated input sizes and requirements.

## 12. Suggested solutions for the exercises

Use these only after making your own attempt. The implementations define one reasonable contract for each exercise.

```python
from collections import defaultdict


def count_failed(statuses):
    count = 0

    for status in statuses:
        if status == "failed":
            count += 1

    return count


def duplicate_hosts(hostnames):
    seen = set()
    duplicates = set()

    for hostname in hostnames:
        if hostname in seen:
            duplicates.add(hostname)
        else:
            seen.add(hostname)

    return duplicates


def group_services(records):
    groups = defaultdict(list)

    for record in records:
        groups[record["status"]].append(record["name"])

    return dict(groups)


def first_above(values, threshold):
    for index, value in enumerate(values):
        if value > threshold:
            return index

    return -1


def largest_k(priorities, k):
    if k <= 0:
        return []

    return sorted(priorities, reverse=True)[:k]


assert count_failed(["healthy", "failed", "failed"]) == 2
assert count_failed([]) == 0

assert duplicate_hosts(
    ["node-a", "node-b", "node-a"]
) == {"node-a"}

assert group_services([
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "database", "status": "healthy"},
]) == {
    "healthy": ["api", "database"],
    "failed": ["worker"],
}

assert first_above([40, 65, 81, 55], 70) == 2
assert first_above([40, 65, 55], 70) == -1

assert largest_k([4, 9, 1, 7], 2) == [9, 7]
assert largest_k([4, 9], 0) == []
assert largest_k([4, 9], 5) == [9, 4]
```

### Complexity discussion

| Exercise | Time | Auxiliary space (excluding returned result) |
|---|---:|---:|
| Count failed jobs | (O(n)) | (O(1)) |
| Find duplicate hostnames | Expected (O(n)) | (O(n)) |
| Group services by status | Expected (O(n)) | (O(n)) |
| Find first value above threshold | (O(n)) worst case | (O(1)) |
| Return largest (k) using sorting | (O(n \log n)) | Depends on sorting implementation and returned result; do not count the returned slice as auxiliary space |

These are high-level analyses under the usual expected hash-table operation assumption. The final exercise intentionally uses sorting as a clear baseline; a heap-based solution is a useful follow-up when (k) is small relative to (n).

## 13. Readiness checklist

Before moving on, check that you can do the following without copying examples:

- [ ] Write a function with parameters and a meaningful return value.
- [ ] Use conditions and loops correctly.
- [ ] Traverse a list with and without indexes.
- [ ] Manipulate strings without assuming they are mutable.
- [ ] Count values with a dictionary or `Counter`.
- [ ] Detect duplicates with a set.
- [ ] Explain list, dictionary, set, and tuple use cases.
- [ ] Use `deque` for a FIFO queue.
- [ ] Explain what a min-heap exposes at its root.
- [ ] Use `bisect_left` and distinguish an insertion point from a confirmed match.
- [ ] Handle expected exceptions deliberately.
- [ ] Write tests for empty, boundary, and duplicate inputs.
- [ ] Explain time and auxiliary-space complexity.
- [ ] Explain the difference between `sort()` and `sorted()`.

If several items are unfamiliar, revisit the relevant foundation lesson before moving to pattern recognition.

---

## 14. Official Python references

These official documentation pages were consulted for the language and standard-library behavior in this guide:

- [The Python Tutorial — More Control Flow Tools](https://docs.python.org/3/tutorial/controlflow.html) — conditions, loops, functions, and related control flow.
- [The Python Tutorial — Data Structures](https://docs.python.org/3/tutorial/datastructures.html) — list operations and using lists as stacks and queues.
- [Built-in Types](https://docs.python.org/3/library/stdtypes.html) — built-in sequence, mapping, set, and other types.
- [collections — Container datatypes](https://docs.python.org/3/library/collections.html) — `Counter`, `defaultdict`, and `deque`.
- [heapq — Heap queue algorithm](https://docs.python.org/3/library/heapq.html) — heap and priority-queue operations.
- [bisect — Array bisection algorithm](https://docs.python.org/3/library/bisect.html) — insertion-point search functions.
- [Built-in Exceptions](https://docs.python.org/3/library/exceptions.html) — Python exception hierarchy and built-in exceptions.

The DevOps examples and interview exercises are original teaching examples for this repository. They do not describe a specific vendor product or production system.

---

## Navigation

- [Back to DSA Interview Preparation](../README.md)
- [Previous: 01 Interview Roadmap](../01%20Interview%20Roadmap/README.md)
- [DSA for DevOps — Python Basics](../../DSA%20for%20DevOps/01%20Python%20Basics%20for%20DSA/README.md)
- [Complete DSA — Foundations](../../Complete%20DSA/01%20Foundations/README.md)
- [Complete DSA — Interview Patterns](../../Complete%20DSA/17%20Interview%20Patterns/README.md)

**Next:** 03 Pattern Recognition.
