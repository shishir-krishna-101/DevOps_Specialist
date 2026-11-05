# 03 — Minimum, Maximum, and Search

A DevOps script often receives a collection of measurements or records and needs to answer simple questions:

- Which sample has the highest CPU value?
- What is the lowest response-time measurement in this test dataset?
- Does a requested server exist in the inventory?
- At which position does a particular service appear?
- Which service record has the largest memory value?

These are beginner-friendly DSA problems. They teach you to compare values, keep track of a candidate answer, and inspect a collection until you find what you need.

> **Source policy:** Python built-in behavior in this lesson is based on the official Python documentation linked at the end. The hostnames, service records, metrics, and thresholds are invented teaching data. They are not production monitoring recommendations.

---

## 1. Understand the task

Before writing code, identify what the question asks you to return.

| Task | Example input | Expected result |
|---|---|---|
| Find the maximum | `[40, 85, 70, 95]` | `95` |
| Find the minimum | `[40, 85, 70, 95]` | `40` |
| Search for a value | `[10, 20, 30]`, target `20` | Found |
| Find an item's position | `["api", "worker", "db"]`, target `"worker"` | Index `1` |
| Find the record with highest CPU | List of service dictionaries | The matching service record |

The important difference:

- **Minimum/maximum:** inspect values and keep the smallest or largest candidate.
- **Search:** inspect values until a match is found, or until there are no more items to inspect.

## 2. Find the maximum with Python's `max()`

Python provides the built-in `max()` function to return the largest item in an iterable.

```python
cpu_samples = [40, 85, 70, 95, 60]

highest_cpu = max(cpu_samples)

print(highest_cpu)
```

Output:

```text
95
```

Read this as: “Look at the values in `cpu_samples` and return the largest one.”

### Find the minimum with `min()`

The built-in `min()` function returns the smallest item:

```python
response_times = [120, 95, 180, 75, 110]

lowest_time = min(response_times)

print(lowest_time)
```

Output:

```text
75
```

The numbers are illustrative sample values. The code does not measure actual response times.

### Important: empty lists

Calling `max()` or `min()` on an empty iterable without a default raises `ValueError`.

Check the collection first:

```python
cpu_samples = []

if cpu_samples:
    print(max(cpu_samples))
else:
    print("No CPU samples available")
```

Output:

```text
No CPU samples available
```

You can also provide the optional `default` argument when using the iterable form:

```python
cpu_samples = []

highest_cpu = max(cpu_samples, default=None)

print(highest_cpu)
```

Output:

```text
None
```

Use a default that makes sense for your program. Do not silently treat a missing measurement as zero unless zero is genuinely the correct meaning for that application.

## 3. Understand how a maximum scan works

Using `max()` is concise, but it is important to understand the algorithm behind the beginner exercise.

Suppose the input is:

```python
cpu_samples = [40, 85, 70, 95, 60]
```

We can solve the problem manually:

1. Assume the first value is the largest seen so far.
2. Visit the next value.
3. If it is larger than the current largest value, replace the candidate.
4. Continue until every value has been checked.
5. Return the candidate.

### Manual implementation

```python
def find_maximum(numbers):
    if not numbers:
        return None

    largest = numbers[0]

    for number in numbers:
        if number > largest:
            largest = number

    return largest


cpu_samples = [40, 85, 70, 95, 60]

print(find_maximum(cpu_samples))
```

Output:

```text
95
```

### Line-by-line explanation

- `def find_maximum(numbers):` defines a function that accepts a list of numbers.
- `if not numbers:` checks whether the list is empty.
- `return None` defines this lesson's result for empty input.
- `largest = numbers[0]` starts with a real value from the list.
- `for number in numbers:` visits each number.
- `if number > largest:` checks whether the current value is larger.
- `largest = number` updates the candidate when a larger value is found.
- `return largest` returns the final candidate after the loop.

This implementation assumes the input is a list of mutually comparable numeric values. Input validation for untrusted data is a separate concern.

### Dry run

Input: `[40, 85, 70, 95, 60]`

| Step | Current number | Previous largest | Is current larger? | Largest after check |
|---:|---:|---:|---|---:|
| Start | — | — | — | 40 |
| 1 | 40 | 40 | No | 40 |
| 2 | 85 | 40 | Yes | 85 |
| 3 | 70 | 85 | No | 85 |
| 4 | 95 | 85 | Yes | 95 |
| 5 | 60 | 95 | No | 95 |

Final result: `95`.

### Complexity

Let `n` be the number of values.

- **Time:** O(n), because the algorithm visits each value once.
- **Additional space:** O(1), because it keeps only one candidate value in addition to the input.

The empty-list check does not change the overall complexity.

## 4. Find the minimum manually

The minimum algorithm is almost the same. The only important change is the comparison operator.

- Maximum: replace the candidate when the current value is greater.
- Minimum: replace the candidate when the current value is smaller.

```python
def find_minimum(numbers):
    if not numbers:
        return None

    smallest = numbers[0]

    for number in numbers:
        if number < smallest:
            smallest = number

    return smallest


memory_samples = [72, 64, 81, 55, 69]

print(find_minimum(memory_samples))
```

Output:

```text
55
```

### Dry run

| Step | Current number | Previous smallest | Is current smaller? | Smallest after check |
|---:|---:|---:|---|---:|
| Start | — | — | — | 72 |
| 1 | 72 | 72 | No | 72 |
| 2 | 64 | 72 | Yes | 64 |
| 3 | 81 | 64 | No | 64 |
| 4 | 55 | 64 | Yes | 55 |
| 5 | 69 | 55 | No | 55 |

Final result: `55`.

**Complexity:** O(n) time and O(1) additional space.

## 5. Find the service with the highest CPU value

Operational scripts often process records rather than a list of numbers.

Here, each record is a dictionary with a service name and a sample CPU value:

```python
services = [
    {"name": "api", "cpu": 45.0},
    {"name": "worker", "cpu": 86.5},
    {"name": "database", "cpu": 79.0},
    {"name": "scheduler", "cpu": 91.0}
]
```

### Manual solution

```python
def find_service_with_highest_cpu(services):
    if not services:
        return None

    highest_service = services[0]

    for service in services:
        if service["cpu"] > highest_service["cpu"]:
            highest_service = service

    return highest_service


result = find_service_with_highest_cpu(services)

if result is None:
    print("No service records available")
else:
    print(result["name"], result["cpu"])
```

Output:

```text
scheduler 91.0
```

### Understand the algorithm

1. If there are no records, return `None`.
2. Start by treating the first record as the current highest.
3. Compare each record's CPU value with the current candidate's CPU value.
4. Replace the candidate when a larger value is found.
5. Return the complete dictionary, not just its CPU number.

This function assumes every record contains a numeric `"cpu"` value. It does not validate missing keys or unexpected types.

### Dry run

| Service | CPU | Current highest after comparison |
|---|---:|---|
| api | 45.0 | api (45.0) |
| worker | 86.5 | worker (86.5) |
| database | 79.0 | worker (86.5) |
| scheduler | 91.0 | scheduler (91.0) |

**Complexity:**
- Time: O(n), because each record is checked once.
- Additional space: O(1), because only a reference to the current candidate is retained.

### Alternative: use `max()` with `key`

Python's `max()` accepts a `key` function. The key function tells it which value to compare for each item.

```python
highest_service = max(services, key=lambda service: service["cpu"])

print(highest_service["name"])
```

Output:

```text
scheduler
```

For an empty list, provide a default:

```python
highest_service = max(
    services,
    key=lambda service: service["cpu"],
    default=None
)
```

The manual loop is useful for learning the algorithm. The built-in is often shorter when the data is already valid and the comparison rule is clear.

## 6. Search for a value with a loop

A search asks whether a target value exists in a collection.

Example:

```python
hostnames = ["web-01", "web-02", "db-01"]
target = "web-02"
```

### Linear search

Linear search checks items one by one until it finds the target or reaches the end.

```python
def contains_hostname(hostnames, target):
    for hostname in hostnames:
        if hostname == target:
            return True

    return False


hostnames = ["web-01", "web-02", "db-01"]

print(contains_hostname(hostnames, "web-02"))
print(contains_hostname(hostnames, "cache-01"))
```

Output:

```text
True
False
```

### Line-by-line explanation

- The function receives a list and the value to search for.
- The loop visits hostnames in their existing order.
- The equality check compares the current hostname with the target.
- If they match, return `True` immediately.
- If the loop finishes without a match, return `False`.

### Dry run: target is `"db-01"`

| Iteration | Current hostname | Equals target? | Action |
|---:|---|---|---|
| 1 | web-01 | No | Continue |
| 2 | web-02 | No | Continue |
| 3 | db-01 | Yes | Return True |

### Complexity

For a list containing `n` items:

- **Time:** O(n) worst case. The target may be last or may not exist.
- **Additional space:** O(1). The function stores no result collection.

If the target is the first item, the function returns after one comparison.

## 7. Search for an item's index

Sometimes you need the position of the matching value, not only True or False.

```python
def find_hostname_index(hostnames, target):
    for index, hostname in enumerate(hostnames):
        if hostname == target:
            return index

    return -1


hostnames = ["web-01", "web-02", "db-01"]

print(find_hostname_index(hostnames, "web-02"))
print(find_hostname_index(hostnames, "cache-01"))
```

Output:

```text
1
-1
```

In this example, `-1` is our chosen “not found” result. It is a repository design decision for this exercise, not a special return value automatically provided by Python's loop.

### Why use `enumerate()`?

It gives the loop both the index and the value:

```python
for index, hostname in enumerate(hostnames):
    print(index, hostname)
```

Remember that Python list indexes start at zero.

### What if a value occurs more than once?

This function returns the index of the first match because it returns immediately.

```python
hostnames = ["web-01", "db-01", "web-01"]

print(find_hostname_index(hostnames, "web-01"))
```

Output:

```text
0
```

If the task asks for every matching position, you must collect the indexes instead of returning immediately.

## 8. Search with Python's `in` and `index()`

For a simple membership check, Python's `in` operator is concise:

```python
hostnames = ["web-01", "web-02", "db-01"]

print("web-02" in hostnames)
print("cache-01" in hostnames)
```

Output:

```text
True
False
```

If you need the first matching index, lists provide an `index()` method:

```python
hostnames = ["web-01", "web-02", "db-01"]

print(hostnames.index("db-01"))
```

Output:

```text
2
```

Important: `list.index()` raises `ValueError` if the value is not present. Check membership first if absence is expected:

```python
target = "cache-01"

if target in hostnames:
    print(hostnames.index(target))
else:
    print("Hostname not found")
```

The manual linear search in the previous section is still valuable because it shows exactly how a search works.

## 9. Search for a service record by name

Now search a list of dictionaries:

```python
services = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "database", "status": "healthy"}
]
```

Task: return the first record whose name matches the requested name. Return `None` if no record matches.

```python
def find_service(services, target_name):
    for service in services:
        if service["name"] == target_name:
            return service

    return None


result = find_service(services, "worker")

if result is None:
    print("Service not found")
else:
    print(result)
```

Output:

```text
{'name': 'worker', 'status': 'failed'}
```

This example assumes every record contains a valid `"name"` field. It returns the first matching record if duplicate names are present.

**Complexity:**
- Time: O(n) worst case.
- Additional space: O(1), excluding the input records.

## 10. Common beginner mistakes

| Mistake | Why it is a problem | Better habit |
|---|---|---|
| Starting the maximum at zero | All input values might be negative | Start with the first actual item after checking for empty input |
| Starting the minimum at zero | All input values might be positive | Start with the first actual item |
| Forgetting the empty-list case | Indexing the first item or calling min/max can fail | Decide and document the empty-input result |
| Using `>=` or `<=` without thinking about ties | It can change which matching record is retained | Decide whether the first or last tied item should win |
| Returning inside a search loop before checking all values | It is correct for first-match search, but not for collecting all matches | Match the return strategy to the task |
| Assuming `index()` returns a special “not found” value | It raises `ValueError` when absent | Check membership or handle the exception |
| Assuming an unsorted list supports binary search | Binary search relies on sorted input | Use linear search unless ordering is established |
| Comparing incompatible values | Some mixed values cannot be ordered | Validate data types before comparing |

## 11. Practice exercises

Try these without looking at the suggested solutions.

### Exercise 1 — Highest measurement

Write `find_maximum()` that returns the largest number in a list. Return `None` for an empty list.

### Exercise 2 — Lowest measurement

Write `find_minimum()` that returns the smallest number. Return `None` for an empty list.

### Exercise 3 — Does the server exist?

Write `server_exists(hostnames, target)` that returns True if the target is in the list and False otherwise.

### Exercise 4 — Find the server position

Write `find_server_index(hostnames, target)` that returns the first matching index, or `-1` if no match exists.

### Exercise 5 — Find the highest CPU service

Given a list of dictionaries with `"name"` and `"cpu"`, return the record with the highest CPU value. Return `None` for an empty list.

### Exercise 6 — Find every matching server position

Given a list that may contain duplicate hostnames, return a list of every index where the target occurs.

### Exercise 7 — Find the shortest response-time sample

Given a non-empty list of numeric sample values, return the smallest value. Then decide what your function should return for an empty list and document that choice.

### Exercise 8 — Search for a failed service

Given service records, return the first record whose status is `"failed"`. Return `None` if no such record exists.

---

## Suggested solutions

<details>
<summary>Exercise 1 — Highest measurement</summary>

```python
def find_maximum(numbers):
    if not numbers:
        return None

    largest = numbers[0]

    for number in numbers:
        if number > largest:
            largest = number

    return largest


assert find_maximum([4, 9, 2, 7]) == 9
assert find_maximum([-8, -3, -10]) == -3
assert find_maximum([]) is None
```

</details>

<details>
<summary>Exercise 2 — Lowest measurement</summary>

```python
def find_minimum(numbers):
    if not numbers:
        return None

    smallest = numbers[0]

    for number in numbers:
        if number < smallest:
            smallest = number

    return smallest


assert find_minimum([4, 9, 2, 7]) == 2
assert find_minimum([8, 3, 10]) == 3
assert find_minimum([]) is None
```

</details>

<details>
<summary>Exercise 3 — Does the server exist?</summary>

```python
def server_exists(hostnames, target):
    for hostname in hostnames:
        if hostname == target:
            return True

    return False


assert server_exists(["web-01", "db-01"], "db-01") is True
assert server_exists(["web-01", "db-01"], "cache-01") is False
assert server_exists([], "web-01") is False
```

</details>

<details>
<summary>Exercise 4 — Find the server position</summary>

```python
def find_server_index(hostnames, target):
    for index, hostname in enumerate(hostnames):
        if hostname == target:
            return index

    return -1


assert find_server_index(["web-01", "db-01"], "db-01") == 1
assert find_server_index(["web-01", "db-01"], "cache-01") == -1
assert find_server_index([], "web-01") == -1
```

</details>

<details>
<summary>Exercise 5 — Find the highest CPU service</summary>

```python
def find_highest_cpu_service(services):
    if not services:
        return None

    highest = services[0]

    for service in services:
        if service["cpu"] > highest["cpu"]:
            highest = service

    return highest


sample = [
    {"name": "api", "cpu": 40},
    {"name": "worker", "cpu": 82},
    {"name": "database", "cpu": 65}
]

assert find_highest_cpu_service(sample)["name"] == "worker"
assert find_highest_cpu_service([]) is None
```

</details>

<details>
<summary>Exercise 6 — Find every matching server position</summary>

```python
def find_all_server_indexes(hostnames, target):
    indexes = []

    for index, hostname in enumerate(hostnames):
        if hostname == target:
            indexes.append(index)

    return indexes


assert find_all_server_indexes(
    ["web-01", "db-01", "web-01"], "web-01"
) == [0, 2]

assert find_all_server_indexes([], "web-01") == []
```

</details>

<details>
<summary>Exercise 7 — Find the shortest response-time sample</summary>

```python
def shortest_sample(samples):
    if not samples:
        return None

    smallest = samples[0]

    for sample in samples:
        if sample < smallest:
            smallest = sample

    return smallest


assert shortest_sample([120, 95, 180, 75]) == 75
assert shortest_sample([]) is None
```

</details>

<details>
<summary>Exercise 8 — Search for a failed service</summary>

```python
def find_failed_service(services):
    for service in services:
        if service["status"] == "failed":
            return service

    return None


sample = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"}
]

assert find_failed_service(sample)["name"] == "worker"
assert find_failed_service([]) is None
```

</details>

## 12. Test your solutions

Assertions make it easy to check normal and boundary cases.

```python
def find_maximum(numbers):
    if not numbers:
        return None

    largest = numbers[0]

    for number in numbers:
        if number > largest:
            largest = number

    return largest


def find_minimum(numbers):
    if not numbers:
        return None

    smallest = numbers[0]

    for number in numbers:
        if number < smallest:
            smallest = number

    return smallest


def find_index(items, target):
    for index, item in enumerate(items):
        if item == target:
            return index

    return -1


assert find_maximum([4, 9, 2, 7]) == 9
assert find_maximum([-8, -3, -10]) == -3
assert find_maximum([]) is None

assert find_minimum([4, 9, 2, 7]) == 2
assert find_minimum([8, 3, 10]) == 3
assert find_minimum([]) is None

assert find_index(["api", "worker"], "worker") == 1
assert find_index(["api", "worker"], "cache") == -1
assert find_index([], "api") == -1

print("All checks passed")
```

When testing your own solutions, include:

- empty input;
- one item;
- all values equal;
- negative numbers, if the input domain permits them;
- target at the beginning;
- target at the end;
- target absent;
- duplicate target values;
- tied maximum or minimum values.

## 13. Lesson completion checklist

- [ ] Explain the difference between minimum, maximum, and search.
- [ ] Use `max()` and `min()` on non-empty collections.
- [ ] Explain what happens when those functions receive an empty iterable without a default.
- [ ] Find a maximum manually using a loop.
- [ ] Find a minimum manually using a loop.
- [ ] Search for a value with linear search.
- [ ] Return the index of the first matching value.
- [ ] Search a list of dictionaries by a field.
- [ ] Explain why linear search is O(n) in the worst case.
- [ ] Explain why the manual min/max scan uses O(1) additional space.
- [ ] Test empty input, missing targets, duplicates, and ties.

Do not move to the next lesson until you can explain why each loop updates or returns its result.

---

## Official Python documentation

1. [Built-in Functions — `max()`](https://docs.python.org/3/library/functions.html#max) — returns the largest item from an iterable; documents `key` and `default`.
2. [Built-in Functions — `min()`](https://docs.python.org/3/library/functions.html#min) — returns the smallest item from an iterable; documents `key` and `default`.
3. [Built-in Types — List methods](https://docs.python.org/3/tutorial/datastructures.html#more-on-lists) — documents `list.index()` and other list methods.
4. [More Control Flow Tools — for statements](https://docs.python.org/3/tutorial/controlflow.html#for-statements) — explains iteration over sequence items.
5. [Built-in Functions](https://docs.python.org/3/library/functions.html) — reference for Python built-ins.

The documentation verifies Python language and built-in behavior. The O(n) and O(1) statements describe the algorithms shown in this lesson, under the stated assumptions.

---

**Previous:** [02 — Lists and Iteration](../02%20Lists%20and%20Iteration/README.md)

**Next:** [04 — Counting with Loops](../04%20Counting%20with%20Loops/README.md) — count success and failure events using a simple loop.
