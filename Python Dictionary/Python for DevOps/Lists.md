# Lists

Lists are Python's general-purpose mutable sequence type. In DevOps automation, they are useful for ordered collections such as service names, hosts, deployment targets, command arguments, and parsed records.

The goal is to understand list creation, indexing, slicing, mutation, iteration, copying, sorting, and the situations where another collection type is a better fit.

---

## 1. Creating lists

A list can contain zero or more objects.

```python
services = ["api", "worker", "scheduler"]
empty = []
ports = [8080, 9090, 3000]
```

Lists can contain values of different types:

```python
item = ["api", 8080, True]
```

For maintainable automation, prefer collections whose elements have a clear operational meaning.

---

## 2. Lists are ordered

List elements have positions starting at zero.

```python
services = ["api", "worker", "scheduler"]

print(services[0])
print(services[2])
```

The first element is at index `0`.

Negative indexes count from the end:

```python
print(services[-1])
```

An index outside the valid range raises `IndexError`.

---

## 3. Slicing lists

A slice selects a range of elements.

```python
services = ["api", "worker", "scheduler", "frontend"]

print(services[1:3])
```

The start index is included and the stop index is excluded.

Useful forms include:

```python
services[:2]
services[2:]
services[:]
```

A slice creates a new list rather than being another name for the original list.

---

## 4. Lists are mutable

List elements can be changed after the list is created.

```python
services = ["api", "worker"]
services[1] = "scheduler"
```

The list now contains the new value at that position.

This mutability is useful for accumulating and updating automation state, but it also means that multiple names can observe the same underlying list object.

---

## 5. Adding elements with `append()`

`append()` adds one object to the end of the list.

```python
services = ["api"]
services.append("worker")
```

The list becomes:

```text
["api", "worker"]
```

Use `append()` when the new object should be one element of the list.

---

## 6. Adding multiple elements with `extend()`

`extend()` adds elements from an iterable.

```python
services = ["api"]
services.extend(["worker", "scheduler"])
```

This differs from `append()`:

```python
services = ["api"]
services.append(["worker", "scheduler"])
```

The second example adds the entire list as one element.

Choose based on whether the input represents one item or multiple items.

---

## 7. Inserting at a position

`insert()` adds an item at a specified position.

```python
services = ["api", "scheduler"]
services.insert(1, "worker")
```

Use insertion when position is part of the requirement. If order is not important, a different collection may communicate the intent better.

---

## 8. Removing elements

`remove()` removes the first matching value.

```python
services = ["api", "worker", "worker"]
services.remove("worker")
```

If the value is not present, `remove()` raises `ValueError`.

`pop()` removes and returns an item by index, or the last item when no index is supplied.

```python
services = ["api", "worker"]
last = services.pop()
```

Use `del` when you need to delete an item or slice without needing the removed value.

```python
del services[0]
```

---

## 9. Checking membership

Use `in` and `not in` to test membership.

```python
services = ["api", "worker"]

if "api" in services:
    print("API is configured")
```

For repeated membership checks on a large collection, consider whether a `set` better represents the requirement.

---

## 10. Finding positions with `index()`

`index()` returns the position of the first matching value.

```python
services = ["api", "worker", "scheduler"]
position = services.index("worker")
```

If the value is absent, `ValueError` is raised.

If you only need to know whether an item exists, `in` is clearer than using `index()` and catching an exception.

---

## 11. Counting occurrences

`count()` returns how many times a value occurs.

```python
statuses = ["healthy", "failed", "healthy"]
healthy_count = statuses.count("healthy")
```

For many different values, `collections.Counter` can be more appropriate than repeatedly calling `count()`.

---

## 12. Iterating over lists

Lists are iterable.

```python
services = ["api", "worker", "scheduler"]

for service in services:
    print(service)
```

When both position and value are required, use `enumerate()`.

```python
for position, service in enumerate(services, start=1):
    print(position, service)
```

Prefer direct iteration over manual index management when the index is not part of the requirement.

---

## 13. Sorting lists

`sort()` sorts a list in place.

```python
services = ["worker", "api", "scheduler"]
services.sort()
```

`sorted()` returns a new sorted list and leaves the original iterable unchanged.

```python
services = ["worker", "api", "scheduler"]
sorted_services = sorted(services)
```

The distinction matters when the original ordering is operationally meaningful.

---

## 14. Reverse ordering

A list can be sorted in descending order using `reverse=True`.

```python
ports = [8080, 3000, 9090]
ports.sort(reverse=True)
```

A list can also be reversed by calling `reverse()`, which changes the list in place rather than sorting it.

```python
services = ["api", "worker", "scheduler"]
services.reverse()
```

Reversing and sorting are different operations.

---

## 15. Sorting with a key

The `key` argument allows sorting according to a derived value.

```python
services = ["api", "worker", "scheduler"]
services.sort(key=len)
```

For dictionaries representing resources:

```python
resources = [
    {"name": "api", "priority": 2},
    {"name": "worker", "priority": 1},
]

resources.sort(key=lambda resource: resource["priority"])
```

Use a key function that makes the intended ordering explicit.

---

## 16. List comprehensions

A list comprehension creates a list from an iterable using an expression and optional filtering condition.

```python
services = ["api", "worker", "scheduler"]
short_names = [service for service in services if len(service) <= 5]
```

A simple transformation can be concise:

```python
upper_names = [service.upper() for service in services]
```

Do not force complex operational logic into a comprehension. Use a normal loop when multiple actions, exceptions, logging, or side effects are required.

---

## 17. Nested lists

A list can contain other lists.

```python
matrix = [
    [1, 2],
    [3, 4],
]

print(matrix[0][1])
```

Nested structures can represent grouped configuration or tabular data, but for complex records a list of dictionaries or dedicated data structures may be clearer.

---

## 18. Copying lists

Assignment does not create a new list.

```python
services = ["api", "worker"]
alias = services

alias.append("scheduler")
```

Both names refer to the same list object.

A shallow copy can be created with:

```python
copy_one = services.copy()
copy_two = services[:]
copy_three = list(services)
```

The copy contains the same element references as the original. Nested mutable objects therefore require additional care.

---

## 19. Shallow copies and nested data

Consider:

```python
resources = [["api"], ["worker"]]
copy_resources = resources.copy()
```

The outer list is copied, but the nested lists are shared.

```python
copy_resources[0].append("v2")
```

The nested object is therefore visible through both structures.

If independent nested data is required, use an appropriate deep-copy strategy and understand its trade-offs rather than copying blindly.

---

## 20. Comparing lists

Lists support equality comparison based on their elements and ordering.

```python
["api", "worker"] == ["api", "worker"]
```

These are not equal:

```python
["api", "worker"] == ["worker", "api"]
```

The second list has a different order.

When order should not matter, a list may not be the best representation of the requirement.

---

## 21. Lists and configuration data

Lists are common when configuration contains ordered values.

```python
deployment_targets = [
    "api-1",
    "api-2",
    "api-3",
]

for target in deployment_targets:
    validate_target(target)
```

Do not assume a list is interchangeable with a set. A list preserves order and allows duplicates; a set is designed for unique membership-oriented values.

---

## 22. Lists and command arguments

A list is often a natural representation of separate command arguments before passing them to a subprocess API.

Conceptually:

```python
command = ["tool", "status", "--verbose"]
```

Keeping arguments as separate list elements avoids manually constructing one shell command string when the subprocess interface accepts an argument sequence.

The exact execution behavior belongs to the repository's dedicated subprocess topic.

---

## 23. Avoid modifying a list while iterating over it

Removing items from a list while iterating over that same list can produce skipped elements and confusing control flow.

Prefer filtering into a new list when appropriate:

```python
services = ["api", "scheduler", "worker"]
active = [
    service for service in services
    if service != "scheduler"
]
```

If in-place mutation is required, make the iteration strategy explicit and test it.

---

## 24. Empty lists and truthiness

An empty list is false in a Boolean context.

```python
services = []

if not services:
    print("No services configured")
```

Use this when emptiness itself is the meaningful state.

If you need to distinguish an empty list from a missing value such as `None`, test those states explicitly.

---

## 25. Performance considerations

Lists are excellent general-purpose sequences, but the operation you perform determines whether they are appropriate.

Consider the requirement before choosing a collection:

| Requirement | Possible fit |
|---|---|
| Ordered sequence | `list` |
| Unique membership | `set` |
| Key/value lookup | `dict` |
| Queue-like operations at both ends | `collections.deque` |
| Count repeated values | `collections.Counter` |

These are design choices based on the operation you need, not universal performance rules.

For repeated membership checks, for example, a set can be more appropriate because membership is its primary use case.

---

## 26. Common mistakes

### Mistake 1: Confusing `append()` and `extend()`

`append()` adds one object. `extend()` adds elements from an iterable.

### Mistake 2: Assuming assignment copies a list

`other = original` creates another reference to the same list.

### Mistake 3: Modifying a list while iterating

This can skip elements or produce hard-to-review behavior.

### Mistake 4: Using a list when uniqueness is the real requirement

A set may communicate the requirement better.

### Mistake 5: Using `index()` when membership is all that is required

Use `in` when you only need an existence check.

### Mistake 6: Sorting data accidentally

`sort()` changes the list in place. Use `sorted()` when a new ordered result is required.

### Mistake 7: Overusing list comprehensions

Readable loops are preferable when processing contains substantial logic or side effects.

### Mistake 8: Assuming a shallow copy isolates nested objects

Only the outer list is copied.

### Mistake 9: Treating an empty list and `None` as the same state

They can represent different operational meanings.

---

## 27. Practical DevOps exercises

### Exercise 1 — Service inventory

Create a list of service names and print them with `enumerate()`.

### Exercise 2 — Deployment targets

Build a list of deployment targets and validate each target before processing it.

### Exercise 3 — Filtering

Create a new list containing only resources that satisfy a chosen condition.

### Exercise 4 — Deduplication decision

Given a list containing repeated environments, decide whether the requirement calls for preserving duplicates or converting the values to a set.

### Exercise 5 — Sorting resources

Sort a list of resource dictionaries by a numeric priority using a key function.

### Exercise 6 — Safe removal

Remove invalid resources without mutating the list currently being iterated.

### Exercise 7 — Command arguments

Represent a command and its arguments as a list of strings, keeping each argument separate.

### Exercise 8 — Copying configuration

Create a shallow copy of a configuration list and demonstrate which nested objects remain shared.

### Exercise 9 — Operational state

Write code that distinguishes between `None`, an empty list, and a populated list and assigns a different operational meaning to each.

### Exercise 10 — Collection choice

For several automation requirements, choose between `list`, `set`, `dict`, and `deque`, and explain why.

---

## 28. Knowledge check

You should be able to explain:

1. Why list indexes start at zero.
2. How negative indexes work.
3. What list slicing returns.
4. Why lists are called mutable sequences.
5. The difference between `append()` and `extend()`.
6. The difference between `remove()`, `pop()`, and `del`.
7. When to use `enumerate()`.
8. The difference between `sort()` and `sorted()`.
9. What a list comprehension is appropriate for.
10. Why assignment does not copy a list.
11. What a shallow copy means for nested data.
12. Why modifying a list during iteration can be dangerous.
13. When a set may be a better choice than a list.
14. How a list can represent separate command arguments.
15. Why an empty list and `None` may need different handling.

---

## 29. Completion criteria

You have completed this topic when you can:

- create and index lists confidently;
- use slices correctly;
- mutate lists intentionally;
- choose between `append()` and `extend()`;
- remove elements safely;
- iterate directly or with `enumerate()` when appropriate;
- sort without accidentally changing the original data when that matters;
- write readable list comprehensions;
- understand aliasing and shallow copies;
- avoid unsafe mutation during iteration;
- choose a collection type based on the operational requirement;
- use lists to represent ordered automation data and separate command arguments.

---

## Verified facts vs repository design

### Verified facts

Python documents lists as mutable sequences and documents their indexing, slicing, mutation methods, iteration, comparison, sorting, and related sequence behavior.

### Repository design decisions

This topic emphasizes deployment targets, service inventories, configuration, command arguments, validation, and collection selection because the repository targets DevOps automation. These are teaching choices, not Python language requirements.

### Demonstrations

The service, deployment, configuration, and command examples are intentionally small demonstrations. They do not claim behavior for a particular cloud provider, Kubernetes distribution, CI/CD platform, or deployment tool.

---

## Authoritative references

- Python Tutorial — More on Lists: https://docs.python.org/3/tutorial/datastructures.html
- Python Standard Library — Built-in Types: https://docs.python.org/3/library/stdtypes.html
- Python Standard Library — Sequence Types: https://docs.python.org/3/library/stdtypes.html#sequence-types-list-tuple-range
- Python Standard Library — `collections`: https://docs.python.org/3/library/collections.html
- Python Tutorial — Data Structures: https://docs.python.org/3/tutorial/datastructures.html
