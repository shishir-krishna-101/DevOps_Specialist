# 02 — Lists and Iteration

Lists are one of the most useful Python data structures for beginner DSA. They let us keep an ordered collection of values and process those values one at a time.

For DevOps-flavored examples, imagine that a script receives a list of server names, service status records, job results, CPU measurements, or deployment names.

In this lesson, you will learn to read, traverse, inspect, and build lists. You will also solve practical problems: find the first failed service, collect every failed service, and select records that meet a simple condition.

> **Source policy:** Python list and iteration behavior is based on the official Python documentation linked at the end. All hostnames, service records, and measurements are invented teaching data, not claims about a particular DevOps product.

---

## 1. What is a list?

A list is an ordered, mutable collection. Ordered means that items have positions. Mutable means that you can change the collection after creating it.

```python
services = ["api", "worker", "database"]

print(services)
```

Output:

```text
['api', 'worker', 'database']
```

A list can contain strings, numbers, Boolean values, or other objects:

```python
hostnames = ["web-01", "web-02", "db-01"]
cpu_samples = [42, 65, 71, 58]
health_checks = [True, True, False, True]
```

For DSA exercises, it is often clearer to keep a consistent kind of value in a list.

## 2. Indexing: access an item by its position

Python list positions start at index zero.

```python
services = ["api", "worker", "database"]

print(services[0])  # api
print(services[1])  # worker
print(services[2])  # database
```

| Index | Value |
|---:|---|
| 0 | api |
| 1 | worker |
| 2 | database |

The last valid index is one less than the list length.

```python
services = ["api", "worker", "database"]

print(len(services))                   # 3
print(services[len(services) - 1])     # database
```

### Negative indexes

Negative indexes count from the end:

```python
services = ["api", "worker", "database"]

print(services[-1])  # database
print(services[-2])  # worker
```

### Avoid invalid indexes

A list with three items has no item at index 3. Accessing it raises an IndexError.

An empty list has no valid indexes:

```python
services = []
print(len(services))  # 0
```

Before accessing a position that may not exist, check the list length or design the loop so it only visits existing items.

## 3. Change a list

Lists are mutable, so you can update an item:

```python
services = ["api", "worker", "database"]
services[1] = "scheduler"

print(services)
```

Output:

```text
['api', 'scheduler', 'database']
```

Use `append()` to add one item at the end:

```python
services = ["api", "worker"]
services.append("database")

print(services)
```

Output:

```text
['api', 'worker', 'database']
```

Use `extend()` to add items from another iterable:

```python
services = ["api", "worker"]
new_services = ["database", "scheduler"]

services.extend(new_services)
print(services)
```

Use `remove()` to remove the first matching value. It raises ValueError if the value is not present:

```python
services = ["api", "worker", "database"]
services.remove("worker")

print(services)
```

Use `pop()` to remove and return an item. With no index, it removes the last item:

```python
services = ["api", "worker", "database"]

removed_service = services.pop()

print(removed_service)  # database
print(services)         # ['api', 'worker']
```

Calling `pop()` on an empty list raises IndexError. Check whether the list contains an item before popping if it might be empty.

## 4. Loop through every item

A `for` loop is the most direct way to process each item in a list.

```python
services = ["api", "worker", "database"]

for service in services:
    print(service)
```

Output:

```text
api
worker
database
```

Read the loop in plain language:

1. Take the first item and call it `service`.
2. Run the indented code.
3. Take the next item.
4. Repeat until all items have been visited.

### Example: print every hostname

```python
hostnames = ["web-01", "web-02", "db-01"]

for hostname in hostnames:
    print(f"Checking host: {hostname}")
```

Output:

```text
Checking host: web-01
Checking host: web-02
Checking host: db-01
```

| Iteration | Value assigned to `hostname` | Printed text |
|---:|---|---|
| 1 | web-01 | Checking host: web-01 |
| 2 | web-02 | Checking host: web-02 |
| 3 | db-01 | Checking host: db-01 |

The loop visits three items. For a list of n items, a simple full traversal performs n iterations, so its time complexity is O(n).

## 5. Use indexes with `range()`

Sometimes a problem needs both the position and the value.

```python
services = ["api", "worker", "database"]

for index in range(len(services)):
    print(index, services[index])
```

Output:

```text
0 api
1 worker
2 database
```

The stop value of `range()` is not included. Therefore, `range(len(services))` produces indexes from zero up to, but not including, the list length.

If you only need the values, prefer the simpler form `for service in services`. Use indexes when the position itself matters.

## 6. Use `enumerate()` for index and value

Python provides `enumerate()` to loop over both an item's position and its value.

```python
services = ["api", "worker", "database"]

for index, service in enumerate(services):
    print(index, service)
```

Output:

```text
0 api
1 worker
2 database
```

You can choose a different starting number:

```python
services = ["api", "worker", "database"]

for number, service in enumerate(services, start=1):
    print(number, service)
```

Output:

```text
1 api
2 worker
3 database
```

The list indexes are still zero-based. The `start=1` argument changes the number produced by `enumerate()`; it does not change list indexing.

## 7. Check whether a value is present

Use the `in` operator to test membership:

```python
known_hosts = ["web-01", "web-02", "db-01"]

if "web-02" in known_hosts:
    print("Host is in the inventory")
else:
    print("Host is not in the inventory")
```

Use `not in` for the opposite check:

```python
if "cache-01" not in known_hosts:
    print("Host was not found")
```

For a list, membership checking may need to inspect multiple items before finding a match or determining that no match exists. A full scan is O(n) in the worst case. We will compare lists with sets for membership tasks in a later lesson.

## 8. Work with a list of dictionaries

Operational data is often represented as records. A beginner-friendly way to model a record in Python is a dictionary.

```python
services = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "database", "status": "healthy"}
]
```

The outer structure is a list. Each item inside it is a dictionary.

```python
first_service = services[0]

print(first_service["name"])    # api
print(first_service["status"])  # healthy

for service in services:
    print(service["name"], service["status"])
```

Output from the loop:

```text
api healthy
worker failed
database healthy
```

The loop variable `service` refers to one dictionary at a time. In real automation, validate that incoming records contain the fields and value types your code expects.

## 9. Problem 1 — Find the first failed service

**Task:** Given a list of service records, return the name of the first service whose status is `"failed"`. If there is no failed service, return `None`.

### Step 1: Understand the input

```python
services = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "database", "status": "failed"}
]
```

The expected answer is `"worker"`, because it is the first failed record in list order.

### Step 2: Write the algorithm in plain language

1. Start at the first service.
2. Check its status.
3. If its status is failed, return its name immediately.
4. Otherwise, continue to the next service.
5. If every service was checked and none failed, return `None`.

### Step 3: Implement it

```python
def find_first_failed_service(services):
    for service in services:
        if service["status"] == "failed":
            return service["name"]

    return None


services = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "database", "status": "failed"}
]

result = find_first_failed_service(services)
print(result)
```

Output:

```text
worker
```

### Step 4: Understand the important lines

1. `def find_first_failed_service(services):` defines a function and receives the list.
2. `for service in services:` visits each dictionary in order.
3. `service["status"] == "failed"` checks the current record's status.
4. `return service["name"]` immediately returns the first matching service name.
5. `return None` runs if the loop finishes without finding a match.

Why return `None`? It communicates that no matching service was found. It is different from returning the text `"None"`.

### Step 5: Dry run

| Iteration | Service | Status check | Result |
|---:|---|---|---|
| 1 | api | healthy == failed → False | Continue |
| 2 | worker | failed == failed → True | Return worker |

The third service is not checked. The function already returned the answer.

### Step 6: Complexity

Let n be the number of service records.

- **Time:** O(n) worst case, because all records may need to be checked.
- **Additional space:** O(1) for the scan itself, excluding the input list.

If the first record is failed, the function returns after one check. If no record is failed, it checks all n records.

The example assumes every record is a dictionary with valid `"name"` and `"status"` keys. Handling malformed records is an additional input-validation requirement.

## 10. Problem 2 — Collect all failed services

The previous function stopped at the first failure. Now change the task:

**Task:** Return a new list containing the names of every service whose status is `"failed"`.

```python
def find_all_failed_services(services):
    failed_services = []

    for service in services:
        if service["status"] == "failed":
            failed_services.append(service["name"])

    return failed_services


services = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "database", "status": "failed"},
    {"name": "scheduler", "status": "healthy"}
]

result = find_all_failed_services(services)
print(result)
```

Output:

```text
['worker', 'database']
```

### What changed?

In the first problem, `return` was inside the `if` block. That stopped the function at the first match.

In this problem:

- create an empty result list before the loop;
- append each matching service name;
- return the completed result after the loop.

### Dry run

| Iteration | Service | Failed? | Result list after iteration |
|---:|---|---|---|
| Start | — | — | `[]` |
| 1 | api | No | `[]` |
| 2 | worker | Yes | `["worker"]` |
| 3 | database | Yes | `["worker", "database"]` |
| 4 | scheduler | No | `["worker", "database"]` |

### Complexity

For n records:

- **Time:** O(n), because every record is checked.
- **Additional space:** O(k), where k is the number of failed service names stored in the result. In the worst case, k=n.

## 11. Problem 3 — Find services above a sample CPU threshold

**Task:** Given service records with a numeric `"cpu"` field, return the names whose CPU value is greater than 80.

The threshold is an arbitrary value for this exercise. It is not a production alerting recommendation.

```python
def find_high_cpu_services(services, threshold):
    high_cpu_services = []

    for service in services:
        if service["cpu"] > threshold:
            high_cpu_services.append(service["name"])

    return high_cpu_services


services = [
    {"name": "api", "cpu": 45.0},
    {"name": "worker", "cpu": 86.5},
    {"name": "database", "cpu": 79.0},
    {"name": "scheduler", "cpu": 91.0}
]

result = find_high_cpu_services(services, 80)
print(result)
```

Output:

```text
['worker', 'scheduler']
```

Notice that the condition is `>`, not `>=`. A service with CPU exactly 80 is not included by this rule.

### Try changing the threshold

```python
print(find_high_cpu_services(services, 90))
```

Expected output:

```text
['scheduler']
```

This illustrates an important DSA habit: make the condition precise, especially around boundary values.

## 12. Slicing: take part of a list

A slice selects a range of items:

```python
services = ["api", "worker", "database", "scheduler"]

print(services[0:2])
print(services[1:3])
print(services[:2])
print(services[2:])
```

Output:

```text
['api', 'worker']
['worker', 'database']
['api', 'worker']
['database', 'scheduler']
```

The start index is included and the stop index is excluded.

A slice creates a new list containing the selected items. Keep that in mind when working with large collections or when you need to update the original list.

You can also use a step:

```python
measurements = [10, 20, 30, 40, 50, 60]

print(measurements[::2])
```

Output:

```text
[10, 30, 50]
```

This selects every second item, starting at index zero.

## 13. Common beginner mistakes

| Mistake | Why it is a problem | Better habit |
|---|---|---|
| Using index 1 for the first item | Python indexes start at zero | Start with index 0 or iterate directly |
| Accessing an index that does not exist | It raises IndexError | Check the list length or use a safe loop |
| Using `return` inside a loop when you want every match | The function stops at the first return | Collect matches, then return after the loop |
| Modifying a list while iterating over it | Items can be skipped or iteration can become confusing | Build a separate result list |
| Using `range(len(items))` when no index is needed | Adds unnecessary indexing | Use `for item in items` |
| Confusing `append()` and `extend()` | `append()` adds one object; `extend()` adds items from an iterable | Choose based on the intended result |
| Calling `pop()` on an empty list | It raises IndexError | Check whether an item exists first |
| Assuming a slice changes the original list | A slice produces a new list | Assign the slice if you intend to replace part of the original |

### Why avoid changing a list during iteration?

Removing items from a list while looping over it can make the loop skip values because later items shift positions.

A clearer beginner approach is to create a new list:

```python
statuses = ["healthy", "failed", "healthy", "failed"]

healthy_statuses = []

for status in statuses:
    if status == "healthy":
        healthy_statuses.append(status)

print(healthy_statuses)
```

Output:

```text
['healthy', 'healthy']
```

The original list remains unchanged.

## 14. Practice exercises

Try each exercise yourself first. Write a function where the task asks for a reusable result.

### Exercise 1 — Print each server

Given:

```python
servers = ["web-01", "web-02", "db-01"]
```

Loop through the list and print each server.

### Exercise 2 — Return the number of items

Write a function named `count_servers` that accepts a list and returns how many items it contains. You may use the built-in `len()` function.

### Exercise 3 — Find the first unhealthy service

Given a list of service dictionaries, return the name of the first service whose status is not `"healthy"`. Return `None` if every service is healthy or the list is empty.

### Exercise 4 — Collect failed jobs

Given a list of job dictionaries, return a list of all job names whose status is `"failed"`.

### Exercise 5 — Print positions and values

Use `enumerate()` to print each hostname with a position that starts at 1.

### Exercise 6 — Select a range

Given a list of six measurements, create a new list containing the items at indexes 1, 2, and 3.

### Exercise 7 — Find high measurements

Given a list of numeric CPU measurements and a threshold, return a new list containing only values strictly greater than that threshold.

### Exercise 8 — Handle an empty list

Write a function that returns the last item of a list. If the list is empty, return `None`.

---

## Suggested solutions

<details>
<summary>Exercise 1 — Print each server</summary>

```python
servers = ["web-01", "web-02", "db-01"]

for server in servers:
    print(server)
```

</details>

<details>
<summary>Exercise 2 — Return the number of items</summary>

```python
def count_servers(servers):
    return len(servers)


assert count_servers(["web-01", "web-02"]) == 2
assert count_servers([]) == 0
```

</details>

<details>
<summary>Exercise 3 — Find the first unhealthy service</summary>

```python
def find_first_unhealthy_service(services):
    for service in services:
        if service["status"] != "healthy":
            return service["name"]

    return None
```

</details>

<details>
<summary>Exercise 4 — Collect failed jobs</summary>

```python
def collect_failed_jobs(jobs):
    failed_jobs = []

    for job in jobs:
        if job["status"] == "failed":
            failed_jobs.append(job["name"])

    return failed_jobs
```

</details>

<details>
<summary>Exercise 5 — Print positions and values</summary>

```python
hostnames = ["web-01", "web-02", "db-01"]

for position, hostname in enumerate(hostnames, start=1):
    print(position, hostname)
```

</details>

<details>
<summary>Exercise 6 — Select a range</summary>

```python
measurements = [10, 20, 30, 40, 50, 60]

selected = measurements[1:4]

assert selected == [20, 30, 40]
```

The stop index 4 is excluded.

</details>

<details>
<summary>Exercise 7 — Find high measurements</summary>

```python
def find_above_threshold(measurements, threshold):
    result = []

    for measurement in measurements:
        if measurement > threshold:
            result.append(measurement)

    return result


assert find_above_threshold([40, 85, 70, 95], 80) == [85, 95]
```

</details>

<details>
<summary>Exercise 8 — Handle an empty list</summary>

```python
def get_last_item(items):
    if len(items) == 0:
        return None

    return items[-1]


assert get_last_item(["api", "worker"]) == "worker"
assert get_last_item([]) is None
```

</details>

## 15. Test your solutions

Use assertions to check ordinary and boundary inputs.

```python
def find_all_failed_services(services):
    failed_services = []

    for service in services:
        if service["status"] == "failed":
            failed_services.append(service["name"])

    return failed_services


sample = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "database", "status": "failed"}
]

assert find_all_failed_services(sample) == ["worker", "database"]
assert find_all_failed_services([]) == []
assert find_all_failed_services([
    {"name": "api", "status": "healthy"}
]) == []

print("All checks passed")
```

For your own exercises, test at least:

- an empty list;
- a list with one item;
- no matching values;
- one matching value;
- multiple matching values;
- a match at the beginning and at the end.

The sample functions assume that each record has the keys they access. If data comes from an external system, validate its structure before relying on those keys.

## 16. Lesson completion checklist

Before moving on, make sure you can:

- [ ] Explain what a list is and why it is ordered.
- [ ] Explain why Python list indexes start at zero.
- [ ] Access items using positive and negative indexes.
- [ ] Check the length of a list.
- [ ] Add, update, and remove list items.
- [ ] Loop through a list without using indexes.
- [ ] Use `range()` when positions are needed.
- [ ] Use `enumerate()` to get a position and a value.
- [ ] Check whether a value is in a list.
- [ ] Process a list containing dictionaries.
- [ ] Find the first matching record.
- [ ] Collect every matching record into a new list.
- [ ] Explain the time and additional-space complexity of a simple scan.
- [ ] Test empty, single-item, and multiple-match cases.

If any item is difficult, repeat the dry runs with a smaller list and trace the loop one iteration at a time.

---

## Official Python documentation

1. [Data Structures — More on Lists](https://docs.python.org/3/tutorial/datastructures.html#more-on-lists) — list methods and list operations.
2. [The for statement](https://docs.python.org/3/reference/compound_stmts.html#the-for-statement) — iteration syntax.
3. [The range() function](https://docs.python.org/3/library/stdtypes.html#range) — integer sequences used for iteration.
4. [Built-in Functions](https://docs.python.org/3/library/functions.html) — built-in iteration and length helpers such as `enumerate()` and `len()`.
5. [Built-in Types — Lists](https://docs.python.org/3/library/stdtypes.html#list) — list type behavior.

These sources verify Python language and built-in behavior. The algorithm complexity statements above describe the loops and result storage used in the examples.

---

**Previous:** [01 — Python Basics for DSA](../01%20Python%20Basics%20for%20DSA/README.md)

**Next:** [03 — Minimum, Maximum, and Search](../03%20Minimum%20Maximum%20and%20Search/README.md) — find extreme values and locate a target in a list.
