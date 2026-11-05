# 08 — Sorting

DevOps scripts often collect values that are easier to understand when they are arranged in an order:

- servers ranked by CPU usage;
- deployment jobs sorted by start time or priority;
- services listed alphabetically;
- disk usage measurements arranged from largest to smallest;
- failed tasks placed in a useful report order.

**Sorting** means arranging items according to a chosen ordering rule.

Python provides two main tools for sorting:

- `sorted()` creates and returns a new sorted list.
- `list.sort()` sorts a list in place.

This lesson starts with numbers and strings, then moves to dictionaries representing server records.

> **Scope note:** The data and operational scenarios are fictional learning examples. Sorting a sample CPU value does not measure a real server, and sorting a job list does not schedule or execute jobs.

> **Source policy:** Python sorting behavior described here is based on the official Python documentation linked at the end.

---

## 1. Sort numbers

Start with a list of numbers:

```python
response_times = [240, 120, 310, 95, 180]

sorted_times = sorted(response_times)

print(sorted_times)
```

Output:

```text
[95, 120, 180, 240, 310]
```

By default, values are sorted in ascending order: smallest to largest.

The original list remains unchanged:

```python
print(response_times)
```

Output:

```text
[240, 120, 310, 95, 180]
```

This is useful when you need an ordered view but also want to keep the original input available.

## 2. Sort in descending order

Pass `reverse=True` to `sorted()`:

```python
cpu_usage = [35, 82, 47, 91, 63]

highest_first = sorted(cpu_usage, reverse=True)

print(highest_first)
```

Output:

```text
[91, 82, 63, 47, 35]
```

This arranges the largest value first.

You can also sort in ascending order explicitly with `reverse=False`, which is the default.

## 3. Understand `sorted()` versus `list.sort()`

These two tools are related, but they behave differently.

### `sorted()` creates a new list

```python
servers = ["web-03", "web-01", "web-02"]

result = sorted(servers)

print(result)
print(servers)
```

Output:

```text
['web-01', 'web-02', 'web-03']
['web-03', 'web-01', 'web-02']
```

### `list.sort()` changes the original list

```python
servers = ["web-03", "web-01", "web-02"]

result = servers.sort()

print(servers)
print(result)
```

Output:

```text
['web-01', 'web-02', 'web-03']
None
```

The list has been sorted in place. The method returns `None`; it does not return the sorted list.

### Which one should you choose?

| Requirement | Use |
|---|---|
| Keep the original list unchanged | `sorted(values)` |
| Sort a list in place | `values.sort()` |
| Sort an iterable such as a tuple or set | `sorted(values)` |
| Need a new sorted list for another operation | `sorted(values)` |

For beginner scripts, `sorted()` is often easier to reason about because the original list is preserved.

## 4. Sort strings alphabetically

Strings can be sorted too:

```python
services = ["redis", "nginx", "postgres", "apache"]

result = sorted(services)

print(result)
```

Output:

```text
['apache', 'nginx', 'postgres', 'redis']
```

Python compares strings according to their ordering rules. Uppercase and lowercase characters can affect the result, so a mixed-case list may not appear in the order a person expects.

For a case-insensitive sort, use the string method `casefold` as the key:

```python
service_names = ["Redis", "nginx", "Postgres", "apache"]

result = sorted(service_names, key=str.casefold)

print(result)
```

Output:

```text
['apache', 'nginx', 'Postgres', 'Redis']
```

The original spelling and capitalization remain in the result. The key is used for comparisons.

## 5. Sort using a key

Often you do not want to sort the complete item directly. You want to sort by one property.

Consider a list of server records:

```python
servers = [
    {"hostname": "web-01", "cpu": 72},
    {"hostname": "web-02", "cpu": 35},
    {"hostname": "db-01", "cpu": 91},
    {"hostname": "cache-01", "cpu": 48}
]
```

To sort by CPU usage, use the `key` parameter:

```python
servers_by_cpu = sorted(
    servers,
    key=lambda server: server["cpu"]
)

for server in servers_by_cpu:
    print(server["hostname"], server["cpu"])
```

Output:

```text
web-02 35
cache-01 48
web-01 72
db-01 91
```

### Understand the key function

This expression:

```python
lambda server: server["cpu"]
```

means: take one server record and return its `cpu` value.

Python uses the returned values to determine the sort order. The records themselves are kept in the result.

### Sort highest CPU first

Add `reverse=True`:

```python
servers_by_cpu = sorted(
    servers,
    key=lambda server: server["cpu"],
    reverse=True
)

for server in servers_by_cpu:
    print(server["hostname"], server["cpu"])
```

Output:

```text
db-01 91
web-01 72
cache-01 48
web-02 35
```

This example sorts the sample records by their numeric `cpu` field. It does not define a monitoring threshold or indicate whether a server is unhealthy.

## 6. Sort by more than one field

You can return a tuple from the key function to sort by multiple fields.

Suppose records contain an environment and a hostname:

```python
servers = [
    {"hostname": "web-02", "environment": "production"},
    {"hostname": "web-01", "environment": "staging"},
    {"hostname": "db-01", "environment": "production"},
    {"hostname": "api-01", "environment": "staging"}
]
```

Sort first by environment and then by hostname:

```python
ordered_servers = sorted(
    servers,
    key=lambda server: (
        server["environment"],
        server["hostname"]
    )
)

for server in ordered_servers:
    print(server["environment"], server["hostname"])
```

Output:

```text
production db-01
production web-02
staging api-01
staging web-01
```

Python compares the first tuple element first. If those values are equal, it compares the next element.

This is useful for reports grouped by a primary field and arranged within each group.

## 7. Sort by a dictionary field safely

The previous examples assume every record contains the field used by the key function.

For example, this can raise `KeyError` if a record has no `cpu` key:

```python
# sorted(servers, key=lambda server: server["cpu"])
```

If missing values are valid input, decide explicitly where those records should appear.

For example, place records with a missing CPU field after records with a numeric CPU value:

```python
servers = [
    {"hostname": "web-01", "cpu": 72},
    {"hostname": "web-02"},
    {"hostname": "db-01", "cpu": 91}
]

ordered_servers = sorted(
    servers,
    key=lambda server: (
        "cpu" not in server,
        server.get("cpu", 0)
    )
)

for server in ordered_servers:
    print(server)
```

The first part of the tuple is `False` for records with a CPU field and `True` for records without one. Since `False` sorts before `True`, missing-field records appear last.

The second part sorts records that have the field by numeric CPU value. The default `0` is not used to rank missing records against present values because the first tuple element separates those groups.

**Important:** This is one repository design choice, not a universal rule. In a real report, decide whether missing measurements belong first, last, in a separate section, or should make the input invalid.

## 8. Sort by a key with `min()` and `max()`

Sometimes you do not need the full sorted list. You only need the smallest or largest record.

Use `min()` and `max()` with the same kind of key function:

```python
servers = [
    {"hostname": "web-01", "cpu": 72},
    {"hostname": "web-02", "cpu": 35},
    {"hostname": "db-01", "cpu": 91}
]

lowest_cpu_server = min(servers, key=lambda server: server["cpu"])
highest_cpu_server = max(servers, key=lambda server: server["cpu"])

print(lowest_cpu_server)
print(highest_cpu_server)
```

Output:

```python
{"hostname": "web-02", "cpu": 35}
{"hostname": "db-01", "cpu": 91}
```

Use sorting when you need the complete ranking. Use `min()` or `max()` when you only need one extreme value.

## 9. Sorting is stable

Python sorting is stable. If two records have equal sort keys, their original relative order is preserved.

Example:

```python
jobs = [
    {"job": "backup-a", "priority": 2},
    {"job": "deploy-a", "priority": 1},
    {"job": "backup-b", "priority": 2},
    {"job": "deploy-b", "priority": 1}
]

ordered_jobs = sorted(jobs, key=lambda job: job["priority"])

for job in ordered_jobs:
    print(job["job"], job["priority"])
```

Output:

```text
deploy-a 1
deploy-b 1
backup-a 2
backup-b 2
```

Within priority 1, `deploy-a` remains before `deploy-b`. Within priority 2, `backup-a` remains before `backup-b`.

This is useful when a previous ordering should be retained among records with equal keys.

## 10. Mini project — rank servers by CPU usage

**Task:** Write a function that accepts server records and returns a new list ordered from highest CPU usage to lowest.

### Example input

```python
servers = [
    {"hostname": "web-01", "cpu": 72},
    {"hostname": "web-02", "cpu": 35},
    {"hostname": "db-01", "cpu": 91},
    {"hostname": "cache-01", "cpu": 48}
]
```

### Implementation

```python
def rank_servers_by_cpu(servers):
    return sorted(
        servers,
        key=lambda server: server["cpu"],
        reverse=True
    )


ranked_servers = rank_servers_by_cpu(servers)

for server in ranked_servers:
    print(f"{server['hostname']}: {server['cpu']}%")
```

Output:

```text
db-01: 91%
web-01: 72%
cache-01: 48%
web-02: 35%
```

### Line-by-line explanation

- `def rank_servers_by_cpu(servers):` defines a function that accepts a list of server dictionaries.
- `sorted(` requests a new sorted list.
- `servers,` is the input collection.
- `key=lambda server: server["cpu"],` tells Python to compare the numeric CPU field.
- `reverse=True` places larger values first.
- `return` returns the new sorted list.
- The `for` loop prints each server's hostname and CPU value.

### Dry run

| Server | CPU | Position in descending result |
|---|---:|---:|
| web-01 | 72 | 2 |
| web-02 | 35 | 4 |
| db-01 | 91 | 1 |
| cache-01 | 48 | 3 |

### Tests

```python
def rank_servers_by_cpu(servers):
    return sorted(
        servers,
        key=lambda server: server["cpu"],
        reverse=True
    )


sample = [
    {"hostname": "web-01", "cpu": 72},
    {"hostname": "web-02", "cpu": 35},
    {"hostname": "db-01", "cpu": 91}
]

ranked = rank_servers_by_cpu(sample)

assert [server["hostname"] for server in ranked] == [
    "db-01",
    "web-01",
    "web-02"
]

# sorted() returns a new list; the input order remains unchanged.
assert [server["hostname"] for server in sample] == [
    "web-01",
    "web-02",
    "db-01"
]

assert rank_servers_by_cpu([]) == []

print("All checks passed")
```

Expected output:

```text
All checks passed
```

### Complexity

Let `n` be the number of server records.

- **Time:** O(n log n) for comparison sorting in the general case.
- **Additional space:** Sorting creates a new result list; the exact implementation's auxiliary memory use is an implementation detail. The returned list contains n references to the input records.

This is a sorting demonstration, not a performance benchmark.

## 11. Common mistakes

| Mistake | Why it is a problem | Better habit |
|---|---|---|
| Expecting `sorted()` to change the original list | It returns a new list | Assign the returned value |
| Expecting `list.sort()` to return a list | It returns `None` | Call it separately, then use the original list |
| Sorting records without a key | Dictionaries do not have a general natural ordering for this task | Provide a key function |
| Forgetting `reverse=True` | The default is ascending order | Set the direction explicitly |
| Sorting mixed, incomparable values directly | Python may raise `TypeError` | Normalize or separate values using a deliberate rule |
| Assuming strings sort case-insensitively | Case affects string ordering | Use `key=str.casefold` when appropriate |
| Assuming every record has the requested field | A missing key can raise `KeyError` | Validate fields or define missing-value behavior |
| Assuming equal keys are randomly reordered | Python sorting is stable | Use the stability guarantee when useful |
| Sorting when only the maximum is needed | A full ordering is unnecessary | Consider `max()` or `min()` |

## 12. Practice exercises

Try these before opening the suggested solutions.

### Exercise 1 — Sort numbers

Sort `[8, 3, 10, 1, 5]` from smallest to largest.

### Exercise 2 — Reverse the order

Sort the same values from largest to smallest.

### Exercise 3 — Sort service names

Sort `["redis", "nginx", "postgres"]` alphabetically.

### Exercise 4 — Sort by CPU

Given a list of server dictionaries, sort them by CPU usage from lowest to highest.

### Exercise 5 — Sort by memory

Sort server records by their `memory` field from highest to lowest.

### Exercise 6 — Sort by two fields

Sort job records first by `environment`, then by `job_name`.

### Exercise 7 — Find the highest CPU record

Use `max()` with a key function to find the server with the highest CPU value.

### Exercise 8 — Preserve the original list

Use `sorted()` and verify that the input list has not changed.

### Exercise 9 — Handle a missing field

Sort records with a numeric `cpu` value before records without that field.

---

## Suggested solutions

<details>
<summary>Exercise 1 — Sort numbers</summary>

```python
values = [8, 3, 10, 1, 5]

assert sorted(values) == [1, 3, 5, 8, 10]
```

</details>

<details>
<summary>Exercise 2 — Reverse the order</summary>

```python
values = [8, 3, 10, 1, 5]

assert sorted(values, reverse=True) == [10, 8, 5, 3, 1]
```

</details>

<details>
<summary>Exercise 3 — Sort service names</summary>

```python
services = ["redis", "nginx", "postgres"]

assert sorted(services) == ["nginx", "postgres", "redis"]
```

</details>

<details>
<summary>Exercise 4 — Sort by CPU</summary>

```python
servers = [
    {"hostname": "web-01", "cpu": 72},
    {"hostname": "web-02", "cpu": 35},
    {"hostname": "db-01", "cpu": 91}
]

result = sorted(servers, key=lambda server: server["cpu"])

assert [server["hostname"] for server in result] == [
    "web-02", "web-01", "db-01"
]
```

</details>

<details>
<summary>Exercise 5 — Sort by memory</summary>

```python
servers = [
    {"hostname": "web-01", "memory": 60},
    {"hostname": "db-01", "memory": 85},
    {"hostname": "cache-01", "memory": 40}
]

result = sorted(
    servers,
    key=lambda server: server["memory"],
    reverse=True
)

assert [server["hostname"] for server in result] == [
    "db-01", "web-01", "cache-01"
]
```

</details>

<details>
<summary>Exercise 6 — Sort by two fields</summary>

```python
jobs = [
    {"environment": "production", "job_name": "backup"},
    {"environment": "staging", "job_name": "deploy"},
    {"environment": "production", "job_name": "deploy"}
]

result = sorted(
    jobs,
    key=lambda job: (job["environment"], job["job_name"])
)

assert [
    (job["environment"], job["job_name"]) for job in result
] == [
    ("production", "backup"),
    ("production", "deploy"),
    ("staging", "deploy")
]
```

</details>

<details>
<summary>Exercise 7 — Find the highest CPU record</summary>

```python
servers = [
    {"hostname": "web-01", "cpu": 72},
    {"hostname": "db-01", "cpu": 91}
]

highest = max(servers, key=lambda server: server["cpu"])

assert highest["hostname"] == "db-01"
```

</details>

<details>
<summary>Exercise 8 — Preserve the original list</summary>

```python
values = [3, 1, 2]
result = sorted(values)

assert result == [1, 2, 3]
assert values == [3, 1, 2]
```

</details>

<details>
<summary>Exercise 9 — Handle a missing field</summary>

```python
servers = [
    {"hostname": "web-01", "cpu": 72},
    {"hostname": "web-02"},
    {"hostname": "db-01", "cpu": 91}
]

result = sorted(
    servers,
    key=lambda server: (
        "cpu" not in server,
        server.get("cpu", 0)
    )
)

assert [server["hostname"] for server in result] == [
    "web-01", "db-01", "web-02"
]
```

</details>

## 13. Lesson completion checklist

- [ ] Explain what sorting means.
- [ ] Sort numbers in ascending and descending order.
- [ ] Sort strings.
- [ ] Explain the difference between `sorted()` and `list.sort()`.
- [ ] Explain why `list.sort()` returns `None`.
- [ ] Use `reverse=True`.
- [ ] Sort records with a `key` function.
- [ ] Sort by more than one field using a tuple key.
- [ ] Define what should happen when a sort field is missing.
- [ ] Use `min()` and `max()` when a complete ranking is unnecessary.
- [ ] Explain stable sorting.
- [ ] Test empty input and verify whether the original list changes.

---

## Official Python documentation

1. [Sorting HOW TO](https://docs.python.org/3/howto/sorting.html) — explains `sorted()`, `list.sort()`, key functions, reverse sorting, and stable sorting.
2. [Built-in Function — sorted()](https://docs.python.org/3/library/functions.html#sorted) — reference for the `sorted()` built-in function.
3. [Built-in Types — list.sort()](https://docs.python.org/3/library/stdtypes.html#list.sort) — reference for sorting a list in place.

The official documentation verifies the sorting API and behavior. The server ranking, job ordering, and missing-field handling examples are repository learning exercises.

---

**Previous:** [07 — Strings](../07%20Strings/README.md)

**Next:** [09 — Searching](../09%20Searching/README.md) — find values and records efficiently in operational data.
