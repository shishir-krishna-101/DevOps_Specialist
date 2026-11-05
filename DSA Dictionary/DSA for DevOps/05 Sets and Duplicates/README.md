# 05 — Sets and Duplicates

DevOps automation often works with collections that may contain repeated values:

- a host inventory may list the same hostname more than once;
- two configuration sources may contain overlapping server names;
- a deployment input may repeat a service identifier;
- a script may need to check whether a hostname is already known.

Python sets are useful when the task is about **unique values** or **membership** rather than preserving every occurrence.

In this lesson, you will learn how to create sets, remove duplicates, check membership, compare collections, and find duplicate hostnames.

> **Source policy:** Set behavior and operations described here are based on the official Python documentation linked at the end. All hostnames and records are invented teaching data, not claims about a particular inventory or cloud platform.

---

## 1. What is a set?

A set is an unordered collection that stores no duplicate elements.

Look at this list:

```python
hostnames = ["web-01", "db-01", "web-01", "cache-01", "db-01"]

print(hostnames)
```

Output:

```text
['web-01', 'db-01', 'web-01', 'cache-01', 'db-01']
```

The repeated values are still present in the list.

Now convert it to a set:

```python
unique_hostnames = set(hostnames)

print(unique_hostnames)
```

The set contains only one occurrence of each value. Its printed order may vary because sets are unordered.

For example, the elements are:

- `"web-01"`
- `"db-01"`
- `"cache-01"`

**Important:** A set is not a replacement for every list. Use a list when order and repeated entries matter. Use a set when you need unique values or membership checks.

## 2. Create a set

You can create a set using curly braces with values:

```python
environments = {"development", "staging", "production"}

print(environments)
```

You can also create a set from an existing iterable:

```python
hostnames = ["web-01", "db-01", "web-01"]

unique_hostnames = set(hostnames)
```

### The empty-set detail

Use `set()` to create an empty set:

```python
known_hosts = set()

print(known_hosts)
```

Output:

```text
set()
```

Do not use `{}` for an empty set. In Python, `{}` creates an empty dictionary.

```python
empty_dictionary = {}
empty_set = set()

print(type(empty_dictionary))
print(type(empty_set))
```

Output:

```text
<class 'dict'>
<class 'set'>
```

## 3. Remove duplicate hostnames

A common beginner task is to remove duplicate values from a list.

```python
hostnames = [
    "web-01",
    "db-01",
    "web-01",
    "cache-01",
    "db-01"
]

unique_hostnames = set(hostnames)

print(len(hostnames))
print(len(unique_hostnames))
```

Output:

```text
5
3
```

There are five entries in the original list and three unique hostnames.

### What happens to order?

A set is unordered. Do not rely on the order in which values appear when you print or iterate over a set.

If you need a list of unique values in sorted order, you can sort the set:

```python
hostnames = ["web-01", "db-01", "web-01", "cache-01"]

sorted_unique_hostnames = sorted(set(hostnames))

print(sorted_unique_hostnames)
```

Output:

```text
['cache-01', 'db-01', 'web-01']
```

This example sorts the unique strings alphabetically. Sorting is a separate operation; it does not make the set itself ordered.

If the task is to preserve the first-seen order while removing duplicates, use a different approach:

```python
hostnames = ["web-01", "db-01", "web-01", "cache-01", "db-01"]

seen = set()
unique_in_order = []

for hostname in hostnames:
    if hostname not in seen:
        seen.add(hostname)
        unique_in_order.append(hostname)

print(unique_in_order)
```

Output:

```text
['web-01', 'db-01', 'cache-01']
```

Here the list preserves the order in which each hostname first appeared. The set is used only to remember which values have already been encountered.

## 4. Check whether a hostname exists

Use `in` to test whether an item is in a set.

```python
known_hosts = {"web-01", "web-02", "db-01"}

if "web-02" in known_hosts:
    print("Hostname is known")
else:
    print("Hostname is not known")
```

Output:

```text
Hostname is known
```

Use `not in` to check that a value is absent:

```python
if "cache-01" not in known_hosts:
    print("Hostname is not in this set")
```

A common use is to avoid processing the same identifier twice.

### List membership versus set membership

Both lists and sets support `in`.

```python
host_list = ["web-01", "web-02", "db-01"]
host_set = {"web-01", "web-02", "db-01"}

print("db-01" in host_list)
print("db-01" in host_set)
```

Both expressions return `True`.

For a beginner algorithm comparison:

- A list membership check may scan items one by one, so its worst-case time is O(n).
- Set membership is designed for efficient membership testing. In typical hash-based set usage, lookup is expected O(1), but it is not a universal worst-case guarantee for every input and implementation.

Use a set when membership and uniqueness are the main requirements. Do not convert every list to a set automatically; the data structure should match the task.

## 5. Add and remove set elements

Use `add()` to add one element:

```python
known_hosts = {"web-01", "db-01"}

known_hosts.add("cache-01")
known_hosts.add("web-01")

print(known_hosts)
```

Adding a value that is already present does not create a duplicate.

Use `discard()` to remove an element if it is present. If it is absent, `discard()` does not raise an error:

```python
known_hosts = {"web-01", "db-01"}

known_hosts.discard("db-01")
known_hosts.discard("cache-01")

print(known_hosts)
```

Use `remove()` when you want Python to raise `KeyError` if the element is absent:

```python
known_hosts = {"web-01", "db-01"}

known_hosts.remove("db-01")
```

For beginner scripts where absence is normal, `discard()` is often the simpler choice.

## 6. Find duplicate hostnames

Now solve a practical DSA problem.

**Task:** Given a list of hostnames, return a set containing every hostname that appears more than once.

### Example input

```python
hostnames = [
    "web-01",
    "db-01",
    "web-01",
    "cache-01",
    "db-01",
    "worker-01"
]
```

Expected duplicate values:

```text
{'web-01', 'db-01'}
```

### Algorithm in plain language

1. Create an empty set named `seen`.
2. Create an empty set named `duplicates`.
3. Visit each hostname.
4. If it is already in `seen`, add it to `duplicates`.
5. Otherwise, add it to `seen`.
6. Return the duplicate set.

### Python implementation

```python
def find_duplicates(items):
    seen = set()
    duplicates = set()

    for item in items:
        if item in seen:
            duplicates.add(item)
        else:
            seen.add(item)

    return duplicates


hostnames = [
    "web-01",
    "db-01",
    "web-01",
    "cache-01",
    "db-01",
    "worker-01"
]

result = find_duplicates(hostnames)
print(result)
```

The set's display order can vary, but it contains `"web-01"` and `"db-01"`.

### Line-by-line explanation

- `def find_duplicates(items):` defines a reusable function.
- `seen = set()` stores values encountered so far.
- `duplicates = set()` stores values encountered more than once.
- `for item in items:` processes the input one item at a time.
- `if item in seen:` checks whether the value appeared earlier.
- `duplicates.add(item)` records a repeated value.
- `else: seen.add(item)` remembers a value the first time it appears.
- `return duplicates` returns the set of duplicate values.

### Dry run

| Current item | Already in `seen`? | `seen` after processing | `duplicates` after processing |
|---|---|---|---|
| web-01 | No | web-01 | — |
| db-01 | No | web-01, db-01 | — |
| web-01 | Yes | web-01, db-01 | web-01 |
| cache-01 | No | web-01, db-01, cache-01 | web-01 |
| db-01 | Yes | web-01, db-01, cache-01 | web-01, db-01 |
| worker-01 | No | web-01, db-01, cache-01, worker-01 | web-01, db-01 |

### Complexity

Let:

- `n` be the number of input items;
- `u` be the number of unique input values.

- **Time:** Expected O(n) for typical hash-based set membership and insertion.
- **Additional space:** O(u) for the `seen` set, plus space for the duplicate values.

The expected-time statement depends on normal hash-set behavior. It is not a claim that every possible lookup has a strict O(1) worst-case bound.

## 7. Find hostnames missing from an approved inventory

Suppose one list represents the requested hosts and a set represents the approved hosts.

**Task:** Find requested hostnames that are not in the approved set.

```python
approved_hosts = {"web-01", "web-02", "db-01"}

requested_hosts = [
    "web-01",
    "cache-01",
    "db-01",
    "worker-01"
]

unknown_hosts = []

for hostname in requested_hosts:
    if hostname not in approved_hosts:
        unknown_hosts.append(hostname)

print(unknown_hosts)
```

Output:

```text
['cache-01', 'worker-01']
```

The list keeps the order of the request and would also keep repeated unknown hostnames if the input contained duplicates.

If you need only unique unknown hostnames, store them in a set:

```python
unknown_hosts = set()

for hostname in requested_hosts:
    if hostname not in approved_hosts:
        unknown_hosts.add(hostname)

print(unknown_hosts)
```

Remember that a set does not preserve a display order.

This is a teaching example of comparing two collections. It does not implement a real access-control or authorization policy.

## 8. Compare two inventories with set operations

Python sets support operations that are also used in mathematics.

Suppose two sources provide hostnames:

```python
inventory_a = {"web-01", "web-02", "db-01"}
inventory_b = {"web-02", "db-01", "cache-01"}
```

### Union: values in either set

Use `|` or `union()`:

```python
all_hosts = inventory_a | inventory_b

print(all_hosts)
```

The result contains every unique hostname that appears in either inventory.

### Intersection: values present in both

Use `&` or `intersection()`:

```python
shared_hosts = inventory_a & inventory_b

print(shared_hosts)
```

Result elements: `"web-02"` and `"db-01"`.

### Difference: values in the first set but not the second

Use `-` or `difference()`:

```python
only_in_a = inventory_a - inventory_b
only_in_b = inventory_b - inventory_a

print(only_in_a)
print(only_in_b)
```

- `only_in_a` contains `"web-01"`.
- `only_in_b` contains `"cache-01"`.

### Symmetric difference: values in one set but not both

Use `^` or `symmetric_difference()`:

```python
different_hosts = inventory_a ^ inventory_b

print(different_hosts)
```

The result contains `"web-01"` and `"cache-01"`.

### Summary

| Operation | Symbol | Meaning |
|---|---|---|
| Union | `|` | In either set or both |
| Intersection | `&` | In both sets |
| Difference | `-` | In the left set but not the right |
| Symmetric difference | `^` | In one set but not both |

These operations are useful for learning how to compare two inventories or two collections of identifiers. They do not determine whether a host is actually reachable or healthy.

## 9. Count unique hostnames

You can use a set to find the number of distinct values:

```python
hostnames = [
    "web-01",
    "db-01",
    "web-01",
    "cache-01",
    "db-01"
]

unique_count = len(set(hostnames))

print(unique_count)
```

Output:

```text
3
```

This answers “How many different hostnames appear?” It does not answer “How many total hostname entries are there?” The original list length answers the latter.

## 10. Important limitations and common mistakes

| Mistake | Why it is a problem | Better habit |
|---|---|---|
| Expecting a set to preserve list order | Sets are unordered | Keep a list if order matters |
| Using `{}` for an empty set | It creates an empty dictionary | Use `set()` |
| Expecting duplicate values to remain | A set stores unique elements | Use a list when occurrences matter |
| Trying to access a set by index | Sets are not positional sequences | Iterate over the set or use a list |
| Using `remove()` when an item may be absent | It raises `KeyError` if missing | Use `discard()` or check membership |
| Adding a list or dictionary as a set element | Mutable containers such as lists and dictionaries are not hashable set elements | Store a suitable immutable identifier, such as a string or tuple |
| Converting to a set when duplicates are meaningful | Duplicate occurrences are lost | Keep the original list |
| Assuming set membership has a strict O(1) worst-case guarantee | Expected behavior and worst-case guarantees are different | Describe hash-set lookup as expected O(1) |

### Sets cannot be indexed

This is not valid:

```python
host_set = {"web-01", "db-01"}

# host_set[0]  # Do not use: sets do not support positional indexing
```

Iterate instead:

```python
for hostname in host_set:
    print(hostname)
```

The iteration order is not a stable ordering contract for your application. Do not use a set when the order of processing is important.

## 11. Practice exercises

Try these before reading the suggested solutions.

### Exercise 1 — Remove duplicates

Convert a list of hostnames into a set of unique hostnames.

### Exercise 2 — Count unique values

Write a function that returns the number of distinct items in a list.

### Exercise 3 — Check known hosts

Write a function that returns `True` if a hostname is in a set of known hosts, otherwise `False`.

### Exercise 4 — Find duplicate values

Write a function that returns a set of every value that appears more than once in a list.

### Exercise 5 — Preserve first-seen order

Remove duplicates from a list while keeping the first occurrence of each item in the original order.

### Exercise 6 — Find unknown hosts

Given a set of approved hosts and a list of requested hosts, return a list of requested hosts that are not approved.

### Exercise 7 — Compare two inventories

Return the set of hostnames that are present in both inventory sets.

### Exercise 8 — Find hosts only in one inventory

Return the symmetric difference between two inventory sets.

---

## Suggested solutions

<details>
<summary>Exercise 1 — Remove duplicates</summary>

```python
hostnames = ["web-01", "db-01", "web-01", "cache-01"]

unique_hostnames = set(hostnames)

assert unique_hostnames == {"web-01", "db-01", "cache-01"}
```

</details>

<details>
<summary>Exercise 2 — Count unique values</summary>

```python
def count_unique(items):
    return len(set(items))


assert count_unique(["a", "b", "a", "c"]) == 3
assert count_unique([]) == 0
```

</details>

<details>
<summary>Exercise 3 — Check known hosts</summary>

```python
def is_known_host(known_hosts, hostname):
    return hostname in known_hosts


known_hosts = {"web-01", "db-01"}

assert is_known_host(known_hosts, "web-01") is True
assert is_known_host(known_hosts, "cache-01") is False
```

</details>

<details>
<summary>Exercise 4 — Find duplicate values</summary>

```python
def find_duplicates(items):
    seen = set()
    duplicates = set()

    for item in items:
        if item in seen:
            duplicates.add(item)
        else:
            seen.add(item)

    return duplicates


assert find_duplicates(["a", "b", "a", "c", "b"]) == {"a", "b"}
assert find_duplicates(["a", "b"]) == set()
assert find_duplicates([]) == set()
```

</details>

<details>
<summary>Exercise 5 — Preserve first-seen order</summary>

```python
def unique_in_order(items):
    seen = set()
    result = []

    for item in items:
        if item not in seen:
            seen.add(item)
            result.append(item)

    return result


assert unique_in_order(
    ["web-01", "db-01", "web-01", "cache-01", "db-01"]
) == ["web-01", "db-01", "cache-01"]
```

</details>

<details>
<summary>Exercise 6 — Find unknown hosts</summary>

```python
def find_unknown_hosts(approved_hosts, requested_hosts):
    unknown = []

    for hostname in requested_hosts:
        if hostname not in approved_hosts:
            unknown.append(hostname)

    return unknown


approved = {"web-01", "db-01"}
requested = ["web-01", "cache-01", "db-01"]

assert find_unknown_hosts(approved, requested) == ["cache-01"]
```

</details>

<details>
<summary>Exercise 7 — Compare two inventories</summary>

```python
def find_shared_hosts(inventory_a, inventory_b):
    return inventory_a & inventory_b


assert find_shared_hosts(
    {"web-01", "db-01"},
    {"db-01", "cache-01"}
) == {"db-01"}
```

</details>

<details>
<summary>Exercise 8 — Find hosts only in one inventory</summary>

```python
def find_different_hosts(inventory_a, inventory_b):
    return inventory_a ^ inventory_b


assert find_different_hosts(
    {"web-01", "db-01"},
    {"db-01", "cache-01"}
) == {"web-01", "cache-01"}
```

</details>

## 12. Test the duplicate-finding function

Assertions can compare sets without depending on their display order.

```python
def find_duplicates(items):
    seen = set()
    duplicates = set()

    for item in items:
        if item in seen:
            duplicates.add(item)
        else:
            seen.add(item)

    return duplicates


assert find_duplicates(
    ["web-01", "db-01", "web-01", "cache-01", "db-01"]
) == {"web-01", "db-01"}

assert find_duplicates(["web-01", "db-01"]) == set()
assert find_duplicates(["web-01", "web-01"]) == {"web-01"}
assert find_duplicates([]) == set()

print("All checks passed")
```

Expected output:

```text
All checks passed
```

For your own solutions, test:

- an empty list;
- one item;
- no duplicates;
- every item repeated;
- several different duplicate values;
- values that appear more than twice;
- inputs where original order must be preserved.

## 13. Lesson completion checklist

- [ ] Explain what a set is.
- [ ] Explain why a set does not keep duplicate values.
- [ ] Create a set from a list.
- [ ] Create an empty set using `set()`.
- [ ] Explain why `{}` creates a dictionary.
- [ ] Check membership with `in` and `not in`.
- [ ] Add and remove set elements.
- [ ] Explain why sets do not support positional indexing.
- [ ] Find duplicate values using a `seen` set.
- [ ] Preserve input order while removing duplicates.
- [ ] Compare two sets with union, intersection, difference, and symmetric difference.
- [ ] Explain the difference between total entries and unique entries.
- [ ] Explain expected set membership performance without claiming a strict worst-case guarantee.
- [ ] Test empty input and repeated values.

---

## Official Python documentation

1. [The Python Tutorial — Sets](https://docs.python.org/3/tutorial/datastructures.html#sets) — describes sets as unordered collections without duplicate elements, membership tests, and set operations.
2. [Built-in Types — Set Types](https://docs.python.org/3/library/stdtypes.html#set-types-set-frozenset) — reference for Python set and frozenset behavior.
3. [The Python Tutorial — for statements](https://docs.python.org/3/tutorial/controlflow.html#for-statements) — explains iteration over collection items.

The Python documentation verifies set semantics and operations. The duplicate-detection algorithm and its complexity discussion are the learning examples designed for this repository.

---

**Previous:** [04 — Counting with Loops](../04%20Counting%20with%20Loops/README.md)

**Next:** [06 — Dictionaries](../06%20Dictionaries/README.md) — store and look up operational records using key-value pairs.
