# 06 — Dictionaries

DevOps scripts often work with information that belongs together:

- a hostname and its IP address;
- a service name and its current status;
- a job ID and its result;
- an environment name and its configuration;
- an event status and the number of times it occurred.

A Python **dictionary** stores information as key-value pairs. It is useful when you want to look up a value using a meaningful key.

This lesson introduces dictionaries from the beginning and uses small operational examples. The hostnames, services, and job records are fictional teaching data.

> **Source policy:** Python dictionary behavior in this lesson is based on the official Python documentation linked at the end. The operational examples and exercises are repository learning examples, not claims about a particular DevOps platform.

---

## 1. What is a dictionary?

A dictionary stores pairs in this form:

```python
key: value
```

For example:

```python
service_status = {
    "nginx": "running",
    "postgres": "stopped",
    "redis": "running"
}
```

In this example:

- `"nginx"` is a key.
- `"running"` is the value associated with that key.
- `"postgres"` is another key.
- `"stopped"` is its value.

You can think of the dictionary as a small lookup table:

| Key | Value |
|---|---|
| nginx | running |
| postgres | stopped |
| redis | running |

A list is useful when you want a sequence of items. A dictionary is useful when you want to retrieve a value by key.

## 2. Create a dictionary

Use curly braces with key-value pairs:

```python
server_ip = {
    "web-01": "192.0.2.10",
    "web-02": "192.0.2.11",
    "db-01": "192.0.2.20"
}

print(server_ip)
```

The IP addresses in this example come from documentation-reserved example address space and are only sample values.

You can also create an empty dictionary:

```python
server_metadata = {}
```

Or use the `dict()` constructor:

```python
server_metadata = dict()
```

Both create an empty dictionary.

## 3. Read a value using its key

Use square brackets and the key:

```python
service_status = {
    "nginx": "running",
    "postgres": "stopped",
    "redis": "running"
}

print(service_status["nginx"])
```

Output:

```text
running
```

This is a direct lookup: “What value is stored for the key `nginx`?”

### What if the key does not exist?

Reading a missing key with square brackets raises a `KeyError`:

```python
service_status = {"nginx": "running"}

# This key is not present:
# print(service_status["postgres"])
```

When a key may be missing, use `get()`.

```python
status = service_status.get("postgres")

print(status)
```

Output:

```text
None
```

You can provide a default value:

```python
status = service_status.get("postgres", "unknown")

print(status)
```

Output:

```text
unknown
```

**Beginner rule:**

- Use `dictionary[key]` when the key is expected to exist and a missing key should be treated as an error.
- Use `dictionary.get(key, default)` when a missing key is an expected possibility and you want a fallback value.

## 4. Add a new key or update an existing value

Use assignment:

```python
service_status = {
    "nginx": "running",
    "postgres": "stopped"
}

service_status["redis"] = "running"       # Add a new key
service_status["postgres"] = "running"   # Update an existing key

print(service_status)
```

After these assignments:

- `redis` has been added.
- `postgres` now has the value `running`.

A dictionary has unique keys. Assigning a value to an existing key replaces the value previously associated with that key.

## 5. Check whether a key exists

Use `in` to check dictionary keys:

```python
service_status = {
    "nginx": "running",
    "postgres": "stopped"
}

print("nginx" in service_status)
print("redis" in service_status)
```

Output:

```text
True
False
```

Use `not in` to check that a key is absent:

```python
if "redis" not in service_status:
    print("No status record exists for redis")
```

**Important:** `in` checks keys, not values.

```python
service_status = {"nginx": "running"}

print("nginx" in service_status)    # True: nginx is a key
print("running" in service_status)  # False: running is a value
print("running" in service_status.values())  # True: check values explicitly
```

## 6. Loop through a dictionary

A `for` loop over a dictionary visits its keys:

```python
service_status = {
    "nginx": "running",
    "postgres": "stopped",
    "redis": "running"
}

for service in service_status:
    print(service)
```

Output:

```text
nginx
postgres
redis
```

If you need both the key and its value, use `items()`:

```python
for service, status in service_status.items():
    print(f"{service}: {status}")
```

Output:

```text
nginx: running
postgres: stopped
redis: running
```

In modern Python, dictionaries preserve insertion order. This example prints entries in the order they were inserted. If you need a different order, sort the keys explicitly:

```python
for service in sorted(service_status):
    print(service, service_status[service])
```

## 7. Count job results with a dictionary

Now apply dictionaries to a small DevOps-style task.

**Task:** Given a list of job results, count how many jobs have each status.

### Input

```python
job_results = [
    "success",
    "failed",
    "success",
    "running",
    "failed",
    "success"
]
```

We want a result similar to:

```python
{
    "success": 3,
    "failed": 2,
    "running": 1
}
```

### Think through the algorithm

1. Start with an empty dictionary named `counts`.
2. Visit each status in the list.
3. If the status is already a key, increase its count.
4. If it is not a key, create it with a count of one.
5. Return the dictionary.

### Python implementation

```python
def count_statuses(statuses):
    counts = {}

    for status in statuses:
        counts[status] = counts.get(status, 0) + 1

    return counts


job_results = [
    "success",
    "failed",
    "success",
    "running",
    "failed",
    "success"
]

result = count_statuses(job_results)
print(result)
```

### Line-by-line explanation

- `def count_statuses(statuses):` defines a function that receives a list of status strings.
- `counts = {}` creates an empty dictionary.
- `for status in statuses:` processes each status.
- `counts.get(status, 0)` reads the current count. If the key does not exist, it uses zero.
- `+ 1` adds one for the current occurrence.
- `counts[status] = ...` stores the updated count.
- `return counts` returns the completed frequency dictionary.

### Dry run

| Current status | Previous count | New count | Dictionary after processing |
|---|---:|---:|---|
| success | 0 | 1 | success: 1 |
| failed | 0 | 1 | success: 1, failed: 1 |
| success | 1 | 2 | success: 2, failed: 1 |
| running | 0 | 1 | success: 2, failed: 1, running: 1 |
| failed | 1 | 2 | success: 2, failed: 2, running: 1 |
| success | 2 | 3 | success: 3, failed: 2, running: 1 |

The final dictionary contains three distinct status keys and their counts.

### Complexity

Let `n` be the number of status entries and `k` be the number of distinct statuses.

- **Time:** Expected O(n) for typical hash-based dictionary lookups and updates.
- **Additional space:** O(k) for the count dictionary.

The expected-time description is based on typical hash-table behavior. It is not a strict O(1) worst-case guarantee for every individual dictionary operation.

## 8. Count service states

A list of dictionaries can represent multiple records. Each record can contain fields such as a service name and a status.

```python
services = [
    {"name": "nginx", "status": "healthy"},
    {"name": "postgres", "status": "unhealthy"},
    {"name": "redis", "status": "healthy"},
    {"name": "worker", "status": "unknown"}
]
```

Count healthy and unhealthy services:

```python
def count_service_states(services):
    counts = {
        "healthy": 0,
        "unhealthy": 0,
        "unknown": 0
    }

    for service in services:
        status = service.get("status", "unknown")

        if status not in counts:
            counts[status] = 0

        counts[status] += 1

    return counts


result = count_service_states(services)
print(result)
```

Output:

```python
{
    "healthy": 2,
    "unhealthy": 1,
    "unknown": 1
}
```

The function starts with three familiar categories. If it encounters another status, it creates a counter for that status too.

This is a data-processing example. It does not define what “healthy” means for any particular monitoring system.

## 9. Build a lookup by hostname

Suppose records arrive as a list of dictionaries:

```python
servers = [
    {"hostname": "web-01", "environment": "production"},
    {"hostname": "web-02", "environment": "staging"},
    {"hostname": "db-01", "environment": "production"}
]
```

If you repeatedly need to look up a server by hostname, you can build a dictionary keyed by hostname:

```python
servers_by_hostname = {}

for server in servers:
    hostname = server["hostname"]
    servers_by_hostname[hostname] = server

print(servers_by_hostname["db-01"])
```

Output:

```python
{"hostname": "db-01", "environment": "production"}
```

Now the hostname is the lookup key, and the entire record is the value.

**Important:** This simple example assumes each hostname is unique. If two records have the same hostname, the later assignment replaces the earlier value for that key. Decide how your application should handle duplicate identifiers before building a production inventory.

## 10. Update and delete dictionary entries

Update a field by assigning a new value:

```python
server = {
    "hostname": "web-01",
    "status": "starting"
}

server["status"] = "healthy"
```

Delete an entry using `del`:

```python
server = {
    "hostname": "web-01",
    "status": "healthy",
    "environment": "staging"
}

del server["environment"]

print(server)
```

Output:

```python
{"hostname": "web-01", "status": "healthy"}
```

Deleting a key that does not exist raises `KeyError`. If absence is possible, check the key first or use an appropriate method such as `pop(key, default)` when you also want to retrieve and remove the value.

## 11. Dictionary keys and common mistakes

Dictionary keys must be hashable. Common beginner-friendly key types include strings, integers, and tuples whose elements are themselves hashable.

```python
server_ports = {
    "http": 80,
    "https": 443,
    22: "ssh"
}
```

A list cannot be used as a dictionary key:

```python
# Invalid: a list is mutable and cannot be used as a dictionary key.
# invalid = {["web-01"]: "healthy"}
```

| Mistake | Why it is a problem | Better habit |
|---|---|---|
| Reading a missing key with `d[key]` | Raises `KeyError` | Use `get()` if absence is expected |
| Assuming `in` checks values | It checks keys | Use `value in d.values()` to check values |
| Reusing a key without realizing it | The new value replaces the old value | Validate unique identifiers or define duplicate handling |
| Treating a dictionary as a list | Dictionaries are key-value mappings | Look up by key or iterate over `items()` |
| Using a list as a key | Lists are unhashable | Use an immutable, hashable identifier |
| Assuming every status is known | Input may contain unexpected values | Handle unknown categories explicitly |
| Using `{}` when you mean an empty set | `{}` creates an empty dictionary | Use `set()` for an empty set |

## 12. Practice exercises

Try these without looking at the suggested solutions first.

### Exercise 1 — Create a service lookup

Create a dictionary with three service names and their statuses. Print the status of one service.

### Exercise 2 — Safe lookup

Write a function that returns the status for a requested service. Return `"unknown"` if the service key is absent.

### Exercise 3 — Update a service

Change the status of `"nginx"` from `"starting"` to `"healthy"`.

### Exercise 4 — Count statuses

Given a list of job statuses, return a dictionary containing the count of each status.

### Exercise 5 — Iterate through records

Given a dictionary mapping hostnames to statuses, print each hostname and its status.

### Exercise 6 — Find a server record

Given a list of server dictionaries, build a dictionary keyed by hostname and retrieve one record.

### Exercise 7 — Count service states

Given a list of service dictionaries, count each status. Treat a missing status field as `"unknown"`.

### Exercise 8 — Handle an unexpected status

Update your counting function so it can count a status that was not listed in advance.

---

## Suggested solutions

<details>
<summary>Exercise 1 — Create a service lookup</summary>

```python
services = {
    "nginx": "healthy",
    "postgres": "unhealthy",
    "redis": "healthy"
}

assert services["nginx"] == "healthy"
```

</details>

<details>
<summary>Exercise 2 — Safe lookup</summary>

```python
def get_service_status(services, service_name):
    return services.get(service_name, "unknown")


services = {"nginx": "healthy"}

assert get_service_status(services, "nginx") == "healthy"
assert get_service_status(services, "redis") == "unknown"
```

</details>

<details>
<summary>Exercise 3 — Update a service</summary>

```python
services = {"nginx": "starting"}
services["nginx"] = "healthy"

assert services["nginx"] == "healthy"
```

</details>

<details>
<summary>Exercise 4 — Count statuses</summary>

```python
def count_statuses(statuses):
    counts = {}

    for status in statuses:
        counts[status] = counts.get(status, 0) + 1

    return counts


assert count_statuses(
    ["success", "failed", "success", "running", "failed", "success"]
) == {"success": 3, "failed": 2, "running": 1}

assert count_statuses([]) == {}
```

</details>

<details>
<summary>Exercise 5 — Iterate through records</summary>

```python
statuses = {
    "web-01": "healthy",
    "db-01": "unhealthy"
}

for hostname, status in statuses.items():
    print(f"{hostname}: {status}")
```

</details>

<details>
<summary>Exercise 6 — Find a server record</summary>

```python
def index_servers_by_hostname(servers):
    result = {}

    for server in servers:
        result[server["hostname"]] = server

    return result


servers = [
    {"hostname": "web-01", "status": "healthy"},
    {"hostname": "db-01", "status": "unhealthy"}
]

by_hostname = index_servers_by_hostname(servers)

assert by_hostname["db-01"]["status"] == "unhealthy"
```

This solution assumes hostname keys are unique. Add validation if duplicate hostnames should be rejected.

</details>

<details>
<summary>Exercise 7 — Count service states</summary>

```python
def count_service_states(services):
    counts = {}

    for service in services:
        status = service.get("status", "unknown")
        counts[status] = counts.get(status, 0) + 1

    return counts


services = [
    {"name": "nginx", "status": "healthy"},
    {"name": "db"},
    {"name": "redis", "status": "healthy"}
]

assert count_service_states(services) == {
    "healthy": 2,
    "unknown": 1
}
```

</details>

<details>
<summary>Exercise 8 — Handle an unexpected status</summary>

```python
def count_statuses(statuses):
    counts = {}

    for status in statuses:
        if status not in counts:
            counts[status] = 0

        counts[status] += 1

    return counts


assert count_statuses(["success", "queued", "success"]) == {
    "success": 2,
    "queued": 1
}
```

</details>

## 13. Test the counting function

Use assertions to test normal and edge cases:

```python
def count_statuses(statuses):
    counts = {}

    for status in statuses:
        counts[status] = counts.get(status, 0) + 1

    return counts


assert count_statuses(["success", "failed", "success"]) == {
    "success": 2,
    "failed": 1
}

assert count_statuses([]) == {}
assert count_statuses(["success"]) == {"success": 1}
assert count_statuses(["failed", "failed"]) == {"failed": 2}

print("All checks passed")
```

Expected output:

```text
All checks passed
```

Also test:

- a missing dictionary key;
- an empty dictionary;
- an unexpected status;
- a record missing the expected field;
- repeated keys while building a lookup;
- a list containing no records.

## 14. Lesson completion checklist

- [ ] Explain a dictionary as key-value pairs.
- [ ] Create an empty dictionary and a dictionary with initial values.
- [ ] Read a value by key.
- [ ] Explain the difference between `d[key]` and `d.get(key, default)`.
- [ ] Add a new key and update an existing key.
- [ ] Check key membership with `in`.
- [ ] Explain why `in` checks keys rather than values.
- [ ] Loop through keys and through `items()`.
- [ ] Count job statuses with a dictionary.
- [ ] Read fields from a list of dictionaries.
- [ ] Build a lookup dictionary keyed by hostname.
- [ ] Explain how duplicate keys are handled.
- [ ] Explain why dictionary keys must be hashable.
- [ ] Test empty input and missing keys.
- [ ] Describe expected dictionary lookup performance without claiming a strict worst-case guarantee.

---

## Official Python documentation

1. [The Python Tutorial — Dictionaries](https://docs.python.org/3/tutorial/datastructures.html#dictionaries) — introduces key-value pairs, unique keys, lookup, update, deletion, membership, and iteration.
2. [Built-in Types — Mapping Types: dict](https://docs.python.org/3/library/stdtypes.html#mapping-types-dict) — reference for dictionary operations, key requirements, `get()`, and dictionary views.
3. [Python Glossary — Hashable](https://docs.python.org/3/glossary.html#term-hashable) — explains the hashability requirement for dictionary keys and set elements.

The official documentation verifies Python dictionary behavior. The job counting, service state, and hostname lookup examples are teaching exercises created for this repository.

---

**Previous:** [05 — Sets and Duplicates](../05%20Sets%20and%20Duplicates/README.md)

**Next:** [07 — Strings](../07%20Strings/README.md) — inspect and extract useful information from simple log lines.
