# Small DSA Practice Projects for DevOps with Python

You have learned the basic Python data structures and patterns in this track. Now combine them in small, practical exercises.

This lesson contains six **local, educational projects** built with sample data:

1. Service Status Summary
2. Duplicate Inventory Checker
3. Log Status Counter
4. Metric Report Sorter
5. Simple Job Queue Simulator
6. Service Dependency Walker

Each project is intentionally small. The goal is to practise choosing a data structure, processing input, checking edge cases, and explaining the solution.

> **Important:** These projects do not connect to cloud accounts, Kubernetes clusters, CI/CD systems, monitoring platforms, or production infrastructure. They use invented in-memory data only.

## 1. How to approach each project

Use the same engineering routine for every exercise:

1. Understand the requested result.
2. Identify the input and output.
3. Choose a suitable data structure.
4. Write a small example by hand.
5. Implement the simplest correct function.
6. Test normal and edge-case inputs.
7. Explain the time and extra-space complexity.
8. Consider what production safeguards would be needed before adapting the idea.

The examples below use lists, dictionaries, sets, sorting, queues, and graph traversal from earlier lessons.

## Project 1 — Service Status Summary

### Goal

Given service records, count how many services have each status and list services that are not marked healthy.

### Sample input

~~~python
services = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "database", "status": "healthy"},
    {"name": "scheduler", "status": "degraded"},
]
~~~

### Solution

~~~python
def summarize_services(services):
    status_counts = {}
    unhealthy_services = []

    for service in services:
        status = service["status"]
        status_counts[status] = status_counts.get(status, 0) + 1

        if status != "healthy":
            unhealthy_services.append(service["name"])

    return status_counts, unhealthy_services


services = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "database", "status": "healthy"},
    {"name": "scheduler", "status": "degraded"},
]

counts, unhealthy = summarize_services(services)

print(counts)
print(unhealthy)
~~~

Output:

~~~text
{'healthy': 2, 'failed': 1, 'degraded': 1}
['worker', 'scheduler']
~~~

### How it works

- A dictionary stores the count for each status.
- `get(status, 0)` returns the existing count or zero if the status has not appeared yet.
- A list stores names whose status is not exactly `"healthy"`.
- The function returns both results.

### Complexity

For n service records:
- **Time:** O(n), because each record is processed once.
- **Extra space:** O(k + u), where k is the number of distinct statuses and u is the number of non-healthy services stored in the result.

### Edge cases to consider

- Empty service list.
- All services healthy.
- Every service has a different status.
- A record is missing the `name` or `status` key.

This teaching function expects every record to contain both keys. It does not validate or normalize external input.

### Try it

~~~python
assert summarize_services([]) == ({}, [])
assert summarize_services([
    {"name": "api", "status": "healthy"}
]) == ({"healthy": 1}, [])
~~~

## Project 2 — Duplicate Inventory Checker

### Goal

Find hostnames that appear more than once in an inventory list.

### Sample input

~~~python
hostnames = [
    "web-01",
    "api-01",
    "web-02",
    "web-01",
    "worker-01",
    "api-01",
]
~~~

### Solution

~~~python
def find_duplicates(values):
    seen = set()
    duplicates = set()

    for value in values:
        if value in seen:
            duplicates.add(value)
        else:
            seen.add(value)

    return duplicates


hostnames = [
    "web-01",
    "api-01",
    "web-02",
    "web-01",
    "worker-01",
    "api-01",
]

print(sorted(find_duplicates(hostnames)))
~~~

Output:

~~~text
['api-01', 'web-01']
~~~

### How it works

- `seen` stores values encountered at least once.
- If a value is already in `seen`, it is added to `duplicates`.
- A set ensures each duplicate value appears only once in the result.
- `sorted()` makes the displayed result deterministic.

### Complexity

For n values:
- **Time:** O(n) expected for set membership and insertion, plus O(d log d) to sort d duplicate values for display.
- **Extra space:** O(n) in the worst case for the two sets.

### Edge cases to consider

- Empty list.
- No duplicates.
- Every value is the same.
- Case differences such as `"WEB-01"` and `"web-01"`.

This example compares strings exactly. It does not treat different letter casing as equivalent.

### Try it

~~~python
assert find_duplicates([]) == set()
assert find_duplicates(["a", "b", "c"]) == set()
assert find_duplicates(["a", "a", "a"]) == {"a"}
~~~

## Project 3 — Log Status Counter

### Goal

Count status values from a controlled list of structured log records.

This project deliberately uses dictionaries as input records instead of parsing arbitrary log text. Real log formats need format-specific parsing and validation.

### Sample input

~~~python
log_records = [
    {"message": "request completed", "status": "INFO"},
    {"message": "connection timeout", "status": "ERROR"},
    {"message": "retry started", "status": "WARNING"},
    {"message": "request failed", "status": "ERROR"},
]
~~~

### Solution

~~~python
def count_log_statuses(records):
    counts = {}

    for record in records:
        status = record["status"]
        counts[status] = counts.get(status, 0) + 1

    return counts


log_records = [
    {"message": "request completed", "status": "INFO"},
    {"message": "connection timeout", "status": "ERROR"},
    {"message": "retry started", "status": "WARNING"},
    {"message": "request failed", "status": "ERROR"},
]

print(count_log_statuses(log_records))
~~~

Output:

~~~text
{'INFO': 1, 'ERROR': 2, 'WARNING': 1}
~~~

### How it works

The function reads each record's `status` field and increments the corresponding dictionary count. It does not inspect the message text.

### Complexity

For n records and k distinct statuses:
- **Time:** O(n).
- **Extra space:** O(k) for the status counter.

### Edge cases to consider

- No records.
- One status repeated many times.
- Unexpected but present status values.
- A record without a `status` field.

The function expects every record to contain `status`. Missing fields raise `KeyError`; production input handling should define an explicit policy for malformed records.

### Try it

~~~python
assert count_log_statuses([]) == {}
assert count_log_statuses([
    {"message": "ok", "status": "INFO"},
    {"message": "bad", "status": "ERROR"},
    {"message": "bad again", "status": "ERROR"},
]) == {"INFO": 1, "ERROR": 2}
~~~

## Project 4 — Metric Report Sorter

### Goal

Sort sample host records by CPU usage from highest to lowest.

### Sample input

~~~python
hosts = [
    {"name": "web-01", "cpu": 42},
    {"name": "api-01", "cpu": 87},
    {"name": "worker-01", "cpu": 63},
]
~~~

### Solution

~~~python
def sort_hosts_by_cpu(hosts):
    return sorted(
        hosts,
        key=lambda host: host["cpu"],
        reverse=True,
    )


hosts = [
    {"name": "web-01", "cpu": 42},
    {"name": "api-01", "cpu": 87},
    {"name": "worker-01", "cpu": 63},
]

for host in sort_hosts_by_cpu(hosts):
    print(host["name"], host["cpu"])
~~~

Output:

~~~text
api-01 87
worker-01 63
web-01 42
~~~

### How it works

- `sorted()` returns a new sorted list.
- The `key` function extracts the CPU value from each record.
- `reverse=True` requests descending order.
- The original `hosts` list is not modified by `sorted()`.

### Complexity

For n host records:
- **Time:** O(n log n) for sorting.
- **Extra space:** Sorting uses additional implementation-dependent memory; do not assume this function uses O(1) extra space.

### Edge cases to consider

- Empty host list.
- One host.
- Equal CPU values.
- A record missing its `cpu` field.

This example expects each record to contain a comparable numeric CPU value. It does not validate metric units or data quality.

### Try it

~~~python
assert sort_hosts_by_cpu([]) == []
assert sort_hosts_by_cpu([
    {"name": "a", "cpu": 10},
    {"name": "b", "cpu": 90},
])[0]["name"] == "b"
~~~

## Project 5 — Simple Job Queue Simulator

### Goal

Process job names in the same order they were added to a queue.

This is a local FIFO demonstration. It does not execute jobs or schedule real automation.

### Solution

~~~python
from collections import deque


def process_job_names(job_names):
    pending = deque(job_names)
    processed = []

    while pending:
        job = pending.popleft()
        processed.append(job)

    return processed


jobs = ["build", "test", "package", "deploy"]

print(process_job_names(jobs))
~~~

Output:

~~~text
['build', 'test', 'package', 'deploy']
~~~

### How it works

- `deque(job_names)` creates a queue from the input sequence.
- `popleft()` removes the oldest item.
- The loop ends when no jobs remain.
- The processed list records the order.

### Complexity

For n jobs:
- **Time:** O(n), because each job is removed and appended once.
- **Extra space:** O(n) for the queue and returned processing-order list.

### Edge cases to consider

- Empty job list.
- One job.
- Repeated job names.
- A queue that is already empty.

### Try it

~~~python
assert process_job_names([]) == []
assert process_job_names(["only-job"]) == ["only-job"]
assert process_job_names(["a", "b", "c"]) == ["a", "b", "c"]
~~~

## Project 6 — Service Dependency Walker

### Goal

Represent a tiny service dependency graph and find all services reachable from a starting service.

For this example, an edge means that the first service has a recorded dependency relationship with the second. This is a manually supplied teaching graph, not discovered infrastructure data.

### Sample graph

~~~python
dependencies = {
    "frontend": ["api"],
    "api": ["auth", "database"],
    "auth": ["database"],
    "database": [],
    "worker": ["database"],
}
~~~

### Solution

~~~python
from collections import deque


def reachable_services(graph, start):
    if start not in graph:
        return []

    visited = {start}
    pending = deque([start])
    result = []

    while pending:
        service = pending.popleft()
        result.append(service)

        for dependency in graph[service]:
            if dependency not in visited:
                visited.add(dependency)
                pending.append(dependency)

    return result


dependencies = {
    "frontend": ["api"],
    "api": ["auth", "database"],
    "auth": ["database"],
    "database": [],
    "worker": ["database"],
}

print(reachable_services(dependencies, "frontend"))
~~~

Output:

~~~text
['frontend', 'api', 'auth', 'database']
~~~

The worker is not returned because it cannot be reached by following dependency edges outward from frontend.

### How it works

This is Breadth-First Search:
- A dictionary stores each node's outgoing dependency links.
- A queue stores services waiting to be visited.
- A set prevents visiting a service more than once.
- The result records the traversal order.

### Complexity

For V vertices and E directed edges:
- **Time:** O(V + E) for adjacency-list BFS.
- **Extra space:** O(V) for the visited set, queue, and result list.

### Edge cases to consider

- Unknown starting service.
- Service with no dependencies.
- Cycles in the supplied graph.
- Multiple paths reaching the same dependency.
- Disconnected services.

### Try it

~~~python
assert reachable_services(dependencies, "database") == ["database"]
assert reachable_services(dependencies, "missing") == []
assert reachable_services({
    "a": ["b"],
    "b": ["a"],
}, "a") == ["a", "b"]
~~~

## 7. Run the project checks together

After defining the functions from the six projects, run a compact set of assertions:

~~~python
assert summarize_services([]) == ({}, [])
assert find_duplicates(["node-a", "node-a"]) == {"node-a"}
assert count_log_statuses([]) == {}
assert sort_hosts_by_cpu([]) == []
assert process_job_names(["first", "second"]) == ["first", "second"]
assert reachable_services({"api": ["db"], "db": []}, "api") == ["api", "db"]

print("All project checks passed.")
~~~

Expected output:

~~~text
All project checks passed.
~~~

These assertions are lightweight checks of the expected return values. They are not a complete test suite.

For larger Python projects, the standard-library `unittest` framework provides a testing framework with test cases and test discovery. See the official reference below.

## 8. How to improve these projects responsibly

Once the basic versions work, consider these follow-up improvements:

| Project | Possible improvement | What you would need to decide |
| --- | --- | --- |
| Service Status Summary | Validate each record | Required fields and allowed status values |
| Duplicate Inventory Checker | Normalize hostnames | Whether case or whitespace should be ignored |
| Log Status Counter | Parse log files | Exact log format, encoding, and malformed-line policy |
| Metric Report Sorter | Handle missing metrics | Whether missing values should be rejected or sorted separately |
| Job Queue Simulator | Track job outcomes | What happens when a job fails or is retried |
| Dependency Walker | Detect dependency cycles | Whether cycles are allowed and how they should be reported |

These are repository exercise ideas, not official product recommendations. Choose behavior based on the actual requirements before implementing it.

## 9. Beginner project checklist

- [ ] I can explain the problem each project solves.
- [ ] I can identify the input and output of each function.
- [ ] I can explain why the chosen data structure fits the task.
- [ ] I can trace at least one example by hand.
- [ ] I can run the example and its assertions.
- [ ] I have considered empty input and malformed records.
- [ ] I can explain the time complexity.
- [ ] I can explain the extra space used.
- [ ] I understand which parts are demonstrations rather than production automation.

## 10. Suggested next steps

1. Rebuild each function without looking at the solution.
2. Change the sample data and predict the output before running it.
3. Add tests for one additional edge case per project.
4. Split each project into a separate script only when you are ready to practise Python modules and project structure.
5. Read the Complete DSA track when you want broader algorithmic coverage.

## Official Python references

These sources document the Python language and standard-library features used in the examples. The project scenarios and algorithms are repository teaching material.

- [Python Tutorial — Data Structures](https://docs.python.org/3/tutorial/datastructures.html): lists, dictionaries, sets, and common collection operations.
- [Python Standard Library — Built-in Functions](https://docs.python.org/3/library/functions.html): functions such as `sorted()`, `len()`, and `sum()`.
- [Python Standard Library — `collections.deque`](https://docs.python.org/3/library/collections.html#collections.deque): queue operations used in the examples.
- [Python Standard Library — `unittest`](https://docs.python.org/3/library/unittest.html): standard-library test framework.

## Navigation

- Previous: [13 Graph Basics](../13%20Graph%20Basics/README.md)
- [DSA for DevOps with Python — Roadmap](../README.md)
- Continue learning: [Complete DSA](../../Complete%20DSA/README.md)

---

**Next step:** Choose one project, re-create it without copying, and add at least two tests for edge cases you identify.
