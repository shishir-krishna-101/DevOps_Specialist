# 09 — Searching

DevOps automation often needs to find a particular value or record:

- check whether a hostname is in an inventory;
- find a job by its ID;
- locate a service with a particular status;
- find a metric in a sorted report;
- determine whether a requested package appears in an approved list.

**Searching** means looking through data to find an item that matches a target or condition.

In this lesson, you will learn two beginner search patterns:

1. **Linear search:** check items one by one.
2. **Binary search:** repeatedly narrow the search range in an already sorted list.

You will also learn how Python's `in`, `list.index()`, and `bisect` tools relate to searching.

> **Important:** Binary search requires the data to be sorted according to the same ordering rule used by the search. Do not apply binary search to an arbitrary unsorted list.

> **Source policy:** Python API behavior described here is based on the official documentation linked at the end. The search algorithms are teaching implementations created for this repository.

---

## 1. What does searching mean?

Suppose you have a list of server names:

```python
servers = ["web-01", "web-02", "db-01", "cache-01"]
```

You want to know whether `"db-01"` is present.

A simple way is to check membership:

```python
print("db-01" in servers)
print("worker-01" in servers)
```

Output:

```text
True
False
```

This answers whether a value exists. It does not return the position or the complete matching record.

## 2. Search with `in`

Python's `in` operator is a convenient way to check membership:

```python
approved_hosts = ["web-01", "web-02", "db-01"]

hostname = "web-02"

if hostname in approved_hosts:
    print("Host is listed")
else:
    print("Host is not listed")
```

Output:

```text
Host is listed
```

You can also use `not in`:

```python
if "cache-01" not in approved_hosts:
    print("Host was not found in this list")
```

For a list, membership checking may need to compare the target with each item until a match is found or the list ends. Its worst-case time is O(n), where n is the number of items.

If you only need a membership check for many repeated lookups, a set may be a better fit when the values are hashable. See [05 — Sets and Duplicates](../05%20Sets%20and%20Duplicates/README.md).

## 3. Find the position with `list.index()`

Use `index()` when you want the position of the first matching value:

```python
servers = ["web-01", "web-02", "db-01", "cache-01"]

position = servers.index("db-01")

print(position)
```

Output:

```text
2
```

Python list indexes start at zero, so `"db-01"` is at position 2.

### What if the value is missing?

`list.index()` raises `ValueError` when the value is not present.

```python
servers = ["web-01", "web-02", "db-01"]

# servers.index("worker-01")  # Raises ValueError.
```

Check membership first if a missing value is expected:

```python
target = "worker-01"

if target in servers:
    position = servers.index(target)
    print(position)
else:
    print("Server not found")
```

This performs a membership search and then an index search if found. For a simple one-pass search, you can write your own loop.

## 4. Linear search — check one item at a time

Linear search visits items in sequence until it finds the target or reaches the end.

### Example

```python
servers = ["web-01", "web-02", "db-01", "cache-01"]
target = "db-01"
```

### Algorithm in plain language

1. Start at the first item.
2. Compare the current item with the target.
3. If they match, return the current position.
4. Otherwise, move to the next item.
5. If the list ends without a match, return `-1`.

### Python implementation

```python
def linear_search(items, target):
    for index, item in enumerate(items):
        if item == target:
            return index

    return -1


servers = ["web-01", "web-02", "db-01", "cache-01"]

print(linear_search(servers, "db-01"))
print(linear_search(servers, "worker-01"))
```

Output:

```text
2
-1
```

### Line-by-line explanation

- `def linear_search(items, target):` defines a function with an input collection and a value to find.
- `enumerate(items)` provides each item's index and value.
- `if item == target:` compares the current item with the requested value.
- `return index` stops immediately and returns the first matching position.
- `return -1` means the target was not found after the loop finished.

The function returns the first matching index when duplicates are present.

### Dry run

Searching for `"db-01"`:

| Index | Current item | Matches target? | Action |
|---:|---|---|---|
| 0 | web-01 | No | Continue |
| 1 | web-02 | No | Continue |
| 2 | db-01 | Yes | Return 2 |

The function stops at index 2. It does not need to inspect the remaining item.

### Complexity

Let n be the number of items.

- **Best-case time:** O(1), if the first item matches.
- **Worst-case time:** O(n), if the target is last or absent.
- **Additional space:** O(1), because the loop uses a fixed amount of extra storage.

Linear search works on unsorted data. That makes it a useful starting point.

## 5. Search records using a condition

Often the target is not a simple string. You may want to find a record with a particular field value.

Consider these fictional job records:

```python
jobs = [
    {"job_id": "job-101", "status": "success"},
    {"job_id": "job-102", "status": "failed"},
    {"job_id": "job-103", "status": "running"}
]
```

Find the first failed job:

```python
def find_first_failed_job(jobs):
    for job in jobs:
        if job.get("status") == "failed":
            return job

    return None


failed_job = find_first_failed_job(jobs)

print(failed_job)
```

Output:

```python
{"job_id": "job-102", "status": "failed"}
```

### Explanation

- The loop visits each job record.
- `job.get("status")` safely reads the status field, returning `None` if it is missing.
- If the status equals `"failed"`, the function returns that record.
- If no record matches, the function returns `None`.

This is a linear search over records. It returns the first match in the original list order.

## 6. Binary search — search a sorted list

Linear search may inspect many items. If the input is already sorted, binary search can repeatedly cut the remaining search range in half.

Consider this sorted list:

```python
host_ids = [10, 20, 30, 40, 50, 60, 70, 80, 90]
```

Search for `70`.

Instead of starting at the beginning and checking every value, binary search:

1. Checks the middle item.
2. If the target is smaller, continues in the left half.
3. If the target is larger, continues in the right half.
4. Repeats until it finds the target or the range becomes empty.

### Binary search implementation

```python
def binary_search(items, target):
    left = 0
    right = len(items) - 1

    while left <= right:
        middle = (left + right) // 2

        if items[middle] == target:
            return middle
        elif items[middle] < target:
            left = middle + 1
        else:
            right = middle - 1

    return -1


host_ids = [10, 20, 30, 40, 50, 60, 70, 80, 90]

print(binary_search(host_ids, 70))
print(binary_search(host_ids, 25))
```

Output:

```text
6
-1
```

### Line-by-line explanation

- `left = 0` marks the first possible index.
- `right = len(items) - 1` marks the last possible index.
- `while left <= right:` continues while there is still a search range.
- `middle = (left + right) // 2` calculates the middle index using integer division.
- `items[middle] == target` checks whether the middle value is the target.
- `left = middle + 1` discards the middle and everything to its left when the middle value is smaller.
- `right = middle - 1` discards the middle and everything to its right when the middle value is larger.
- `return -1` means no matching value was found.

### Dry run: search for 70

Input:

```python
[10, 20, 30, 40, 50, 60, 70, 80, 90]
```

| Step | Left index | Right index | Middle index | Middle value | Decision |
|---|---:|---:|---:|---:|---|
| 1 | 0 | 8 | 4 | 50 | 70 is larger; search right |
| 2 | 5 | 8 | 6 | 70 | Match found |

The function returns index 6.

### Dry run: search for 25

| Step | Left index | Right index | Middle index | Middle value | Decision |
|---|---:|---:|---:|---:|---|
| 1 | 0 | 8 | 4 | 50 | 25 is smaller; search left |
| 2 | 0 | 3 | 1 | 20 | 25 is larger; search right |
| 3 | 2 | 3 | 2 | 30 | 25 is smaller; search left |
| 4 | 2 | 1 | — | — | Range is empty; not found |

The function returns `-1`.

### Complexity

For a sorted list of n items:

- **Time:** O(log n), because the remaining search range is approximately halved at each step.
- **Additional space:** O(1), because this iterative implementation uses a fixed number of variables.

### The sorted-input requirement

Binary search is only correct when the input is sorted in the same order used by the comparisons.

This is incorrect:

```python
unsorted_values = [50, 10, 90, 20, 70]

# Do not assume binary_search(unsorted_values, 70) is valid.
```

Sort the data first if that is appropriate:

```python
values = [50, 10, 90, 20, 70]

ordered_values = sorted(values)
position = binary_search(ordered_values, 70)

print(position)
```

Output:

```text
3
```

Notice that the returned index belongs to the sorted list, not the original list. Sorting also has a cost, so if you only need one lookup on a small unsorted list, linear search may be simpler.

## 7. Use Python's `bisect` module

Python includes a standard-library module named `bisect`. It provides tools for finding insertion positions in a sorted list using a bisection algorithm.

Example:

```python
from bisect import bisect_left

values = [10, 20, 30, 40, 50]

position = bisect_left(values, 30)

print(position)
```

Output:

```text
2
```

`bisect_left()` returns the position where the target can be inserted while keeping the list sorted. If equal values already exist, it returns the insertion position before them.

### Find whether a value exists

The result from `bisect_left()` is an insertion position. It is not by itself proof that the target exists.

Check the returned position before indexing:

```python
from bisect import bisect_left

def binary_contains(sorted_items, target):
    position = bisect_left(sorted_items, target)

    return (
        position < len(sorted_items)
        and sorted_items[position] == target
    )


values = [10, 20, 30, 40, 50]

assert binary_contains(values, 30) is True
assert binary_contains(values, 35) is False
assert binary_contains([], 10) is False
```

The input must already be sorted. The check also handles an insertion position at the end of the list without indexing beyond the list.

### Left and right insertion positions

- `bisect_left(items, value)` places the insertion point before existing equal values.
- `bisect_right(items, value)` places the insertion point after existing equal values.

```python
from bisect import bisect_left, bisect_right

values = [10, 20, 20, 20, 30]

print(bisect_left(values, 20))
print(bisect_right(values, 20))
```

Output:

```text
1
4
```

This is useful when you need to locate the range occupied by repeated values in a sorted list.

**Important:** The `bisect` module is documented as a tool for finding insertion points in sorted sequences. The insertion functions such as `insort()` also modify the list; the insertion step can take O(n) time even though the bisection search itself takes O(log n).

## 8. Which search should you choose?

| Situation | Useful approach | Reason |
|---|---|---|
| Small or unsorted list | Linear search or `in` | No sorting prerequisite |
| Need the first matching list index | `list.index()` or linear search | Returns a position |
| Need a matching record | Linear search with a condition | Check a field in each record |
| Many lookups in a sorted list | Binary search or `bisect` | Repeatedly narrows the search range |
| Need an insertion position in sorted data | `bisect_left()` or `bisect_right()` | Finds a suitable position |
| Need repeated membership checks on hashable identifiers | Consider a set | Designed for membership and uniqueness |

Do not choose an algorithm only because its complexity looks better. Consider whether the data is already sorted, how often it changes, how many searches you perform, and whether you need an index or a record.

## 9. Common mistakes

| Mistake | Why it is a problem | Better habit |
|---|---|---|
| Applying binary search to unsorted data | The left/right decisions may discard the target | Sort first or use linear search |
| Forgetting that indexes start at zero | Position calculations become incorrect | Trace indexes carefully |
| Using `list.index()` for a missing value without handling errors | It raises `ValueError` | Check membership or use a controlled loop |
| Returning the middle value instead of its index | The function's result does not match its contract | Decide whether the function returns an index, value, or record |
| Using `right = len(items)` with this inclusive-boundary algorithm | The last index would be out of range | Start with `len(items) - 1` |
| Forgetting to move past the middle index | The loop may never make progress | Use `middle + 1` or `middle - 1` |
| Treating `bisect_left()` as proof of membership | It returns an insertion point even when the value is absent | Check bounds and equality |
| Assuming sorting preserves original positions | Sorted indexes refer to the sorted list | Keep an original-to-sorted mapping if needed |

## 10. Practice exercises

Try these before opening the suggested solutions.

### Exercise 1 — Check membership

Write a function that returns whether a hostname exists in a list.

### Exercise 2 — Find an index

Write a function that returns the first index of a target using a loop. Return `-1` if absent.

### Exercise 3 — Find a failed job

Given a list of job dictionaries, return the first job whose status is `"failed"`. Return `None` if no job matches.

### Exercise 4 — Trace linear search

Trace the search for `"db-01"` in:

```python
["web-01", "db-01", "cache-01"]
```

Write down each index inspected.

### Exercise 5 — Implement binary search

Write an iterative binary search function for a sorted list of integers.

### Exercise 6 — Search an absent value

Use your binary search function to search for `35` in:

```python
[10, 20, 30, 40, 50]
```

Confirm that it returns `-1`.

### Exercise 7 — Use `bisect_left()`

Find the insertion position of `25` in a sorted list.

### Exercise 8 — Handle duplicates with bisect

For a sorted list containing repeated values, find the left and right insertion positions for the repeated value.

---

## Suggested solutions

<details>
<summary>Exercise 1 — Check membership</summary>

```python
def contains_hostname(hostnames, target):
    return target in hostnames


assert contains_hostname(["web-01", "db-01"], "db-01") is True
assert contains_hostname(["web-01", "db-01"], "cache-01") is False
```

</details>

<details>
<summary>Exercise 2 — Find an index</summary>

```python
def linear_search(items, target):
    for index, item in enumerate(items):
        if item == target:
            return index

    return -1


assert linear_search(["web-01", "db-01"], "db-01") == 1
assert linear_search(["web-01", "db-01"], "cache-01") == -1
assert linear_search([], "web-01") == -1
```

</details>

<details>
<summary>Exercise 3 — Find a failed job</summary>

```python
def find_failed_job(jobs):
    for job in jobs:
        if job.get("status") == "failed":
            return job

    return None


jobs = [
    {"job_id": "job-101", "status": "success"},
    {"job_id": "job-102", "status": "failed"}
]

assert find_failed_job(jobs) == {
    "job_id": "job-102",
    "status": "failed"
}
assert find_failed_job([]) is None
```

</details>

<details>
<summary>Exercise 4 — Trace linear search</summary>

The inspected indexes are:

1. Index 0: `"web-01"` — no match.
2. Index 1: `"db-01"` — match.

The returned index is `1`.

</details>

<details>
<summary>Exercise 5 — Implement binary search</summary>

```python
def binary_search(items, target):
    left = 0
    right = len(items) - 1

    while left <= right:
        middle = (left + right) // 2

        if items[middle] == target:
            return middle
        elif items[middle] < target:
            left = middle + 1
        else:
            right = middle - 1

    return -1


assert binary_search([10, 20, 30, 40, 50], 40) == 3
assert binary_search([10, 20, 30, 40, 50], 15) == -1
assert binary_search([], 10) == -1
```

</details>

<details>
<summary>Exercise 6 — Search an absent value</summary>

```python
values = [10, 20, 30, 40, 50]

assert binary_search(values, 35) == -1
```

</details>

<details>
<summary>Exercise 7 — Use bisect_left()</summary>

```python
from bisect import bisect_left

values = [10, 20, 30, 40, 50]

assert bisect_left(values, 25) == 2
```

The insertion position is index 2, before the value 30.

</details>

<details>
<summary>Exercise 8 — Handle duplicates with bisect</summary>

```python
from bisect import bisect_left, bisect_right

values = [10, 20, 20, 20, 30]

assert bisect_left(values, 20) == 1
assert bisect_right(values, 20) == 4
```

The equal values occupy indexes 1, 2, and 3.

</details>

## 11. Test the search functions

```python
def linear_search(items, target):
    for index, item in enumerate(items):
        if item == target:
            return index

    return -1


def binary_search(items, target):
    left = 0
    right = len(items) - 1

    while left <= right:
        middle = (left + right) // 2

        if items[middle] == target:
            return middle
        elif items[middle] < target:
            left = middle + 1
        else:
            right = middle - 1

    return -1


assert linear_search(["a", "b", "c"], "b") == 1
assert linear_search(["a", "b", "c"], "z") == -1
assert linear_search([], "a") == -1

assert binary_search([10, 20, 30, 40, 50], 30) == 2
assert binary_search([10, 20, 30, 40, 50], 35) == -1
assert binary_search([], 10) == -1
assert binary_search([10], 10) == 0
assert binary_search([10], 20) == -1

print("All checks passed")
```

Expected output:

```text
All checks passed
```

Also test:

- a target at the first position;
- a target at the final position;
- an absent target;
- an empty list;
- a list containing one item;
- duplicate values;
- binary search on sorted input only.

## 12. Lesson completion checklist

- [ ] Explain what searching means.
- [ ] Check membership using `in` and `not in`.
- [ ] Find a list index using `index()`.
- [ ] Explain why `index()` raises `ValueError` when absent.
- [ ] Implement linear search using a loop.
- [ ] Search a list of dictionaries by a field.
- [ ] Explain linear search best-case and worst-case time.
- [ ] Explain why binary search requires sorted input.
- [ ] Implement iterative binary search.
- [ ] Trace binary search using left, right, and middle indexes.
- [ ] Explain binary search time complexity.
- [ ] Use `bisect_left()` and `bisect_right()` to find insertion positions.
- [ ] Check membership after obtaining a bisect insertion position.
- [ ] Test empty input, one item, and absent targets.

---

## Official Python documentation

1. [The Python Tutorial — More on Lists](https://docs.python.org/3/tutorial/datastructures.html#more-on-lists) — documents list methods including `index()` and its `ValueError` behavior.
2. [Built-in Types — Common Sequence Operations](https://docs.python.org/3/library/stdtypes.html#common-sequence-operations) — documents sequence membership, indexing, and length operations.
3. [The `bisect` module](https://docs.python.org/3/library/bisect.html) — documents bisection functions for locating insertion points in sorted sequences.

The Python documentation verifies the built-in operations and `bisect` behavior. The linear and binary search functions above are explicit algorithm exercises for this repository.

---

**Previous:** [08 — Sorting](../08%20Sorting/README.md)

**Next:** [10 — Stacks](../10%20Stacks/README.md) — learn the last-in, first-out pattern before queues.
