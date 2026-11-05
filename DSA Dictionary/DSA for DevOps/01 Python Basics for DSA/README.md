# 01 — Python Basics for DSA

Before solving DSA problems with Python, become comfortable reading a small program and understanding how data moves through it.

This lesson introduces only the Python features needed for the first beginner exercises:

- variables and common data types;
- strings and numbers;
- comparisons and conditions;
- lists and indexing;
- loops;
- dictionaries and sets at a basic level;
- functions and return values;
- simple tests with `assert`.

You do not need to memorize every Python feature. Learn the basics, run the examples, change the input, and observe what happens.

> **Source policy:** Python syntax and language behavior in this lesson are based on the official Python documentation linked at the end. The service, host, and job records are invented teaching examples, not descriptions of any specific DevOps product.

---

## 1. What is Python doing in a DSA problem?

A basic DSA exercise usually gives you:

1. **Input** — the information you need to process.
2. **Task** — the rule you must apply to that information.
3. **Output** — the result you must produce.

Example:

- Input: a list of service statuses.
- Task: find the first service whose status is `"failed"`.
- Output: the matching service record, or a clear result when no service failed.

Python gives us syntax and built-in data structures to express the steps.

## 2. Run a small Python program

Create a file named `basics.py` and put this code inside it:

```python
print("Hello, DevOps!")
```

When the program runs, it displays:

```text
Hello, DevOps!
```

- `print()` displays a value.
- The text inside quotes is a string.
- Parentheses are used to call the function.

For the lessons in this track, save examples in a `.py` file and run them with the Python interpreter available in your environment. The exact installation and invocation command can differ by operating system and setup, so use the Python installation guidance appropriate to your environment.

## 3. Variables and basic data types

A variable is a name that refers to a value.

```python
service_name = "payment-api"
cpu_percent = 72.5
retry_count = 3
is_healthy = True

print(service_name)
print(cpu_percent)
print(retry_count)
print(is_healthy)
```

| Variable | Example value | Common Python type | Meaning in this example |
|---|---|---|---|
| `service_name` | `"payment-api"` | `str` | Text |
| `cpu_percent` | `72.5` | `float` | Decimal number |
| `retry_count` | `3` | `int` | Whole number |
| `is_healthy` | `True` | `bool` | True/false value |

You can inspect a value's type with `type()`:

```python
print(type(service_name))
print(type(cpu_percent))
print(type(retry_count))
print(type(is_healthy))
```

### Naming variables

Use names that describe the value:

```python
failed_jobs = 2
server_name = "worker-01"
memory_used_percent = 68.4
```

Avoid unclear names such as `x`, `a1`, or `data2` when a descriptive name makes the code easier to understand.

### Reassigning a variable

A variable can refer to a new value later:

```python
retry_count = 1
retry_count = retry_count + 1

print(retry_count)  # 2
```

The right side is evaluated first. Its result is then assigned to the variable on the left.

## 4. Strings and numbers

Strings represent text and are written inside quotes.

```python
service = "inventory-api"
status = "failed"

message = service + " is " + status
print(message)
```

Output:

```text
inventory-api is failed
```

An f-string is often easier to read when combining text and values:

```python
service = "inventory-api"
cpu_percent = 84.2

message = f"{service} is using {cpu_percent}% CPU"
print(message)
```

Output:

```text
inventory-api is using 84.2% CPU
```

### Basic arithmetic

```python
total_requests = 120
failed_requests = 6

successful_requests = total_requests - failed_requests
failure_rate = failed_requests / total_requests * 100

print(successful_requests)  # 114
print(failure_rate)         # 5.0
```

Important operators:

| Operator | Meaning | Example |
|---|---|---|
| `+` | Addition | `4 + 2` gives `6` |
| `-` | Subtraction | `4 - 2` gives `2` |
| `*` | Multiplication | `4 * 2` gives `8` |
| `/` | Division | `5 / 2` gives `2.5` |
| `//` | Floor division | `5 // 2` gives `2` |
| `%` | Remainder | `5 % 2` gives `1` |
| `**` | Power | `2 ** 3` gives `8` |

Do not divide by zero. A real script should validate values before performing a division when zero is possible.

## 5. Comparisons and conditions

A comparison checks a relationship and produces a Boolean value: `True` or `False`.

```python
cpu_percent = 82

print(cpu_percent > 80)   # True
print(cpu_percent == 80)  # False
print(cpu_percent < 80)   # False
```

Common comparison operators:

| Operator | Meaning |
|---|---|
| `==` | Equal to |
| `!=` | Not equal to |
| `>` | Greater than |
| `<` | Less than |
| `>=` | Greater than or equal to |
| `<=` | Less than or equal to |

Use `if` to run code only when a condition is true:

```python
cpu_percent = 82

if cpu_percent > 80:
    print("CPU usage is above the example threshold")
```

Output:

```text
CPU usage is above the example threshold
```

Use `else` for the alternative path:

```python
service_status = "healthy"

if service_status == "failed":
    print("Investigate this service")
else:
    print("This example did not identify a failed service")
```

Use `elif` when there are multiple conditions:

```python
cpu_percent = 82

if cpu_percent >= 90:
    print("Very high in this example")
elif cpu_percent >= 75:
    print("Elevated in this example")
else:
    print("Below the example thresholds")
```

The thresholds here are arbitrary teaching values. They are not universal operational alerting recommendations.

### A common beginner mistake: `=` vs `==`

- `=` assigns a value.
- `==` compares two values.

```python
status = "failed"          # assignment

if status == "failed":     # comparison
    print("Failure found")
```

## 6. Lists: keeping multiple values together

A list stores an ordered collection of items.

```python
services = ["api", "worker", "database"]

print(services)
print(services[0])  # api
print(services[1])  # worker
print(services[2])  # database
```

Python list indexes start at zero:

| Index | Value |
|---:|---|
| 0 | `"api"` |
| 1 | `"worker"` |
| 2 | `"database"` |

You can add an item with `append()`:

```python
services = ["api", "worker"]
services.append("database")

print(services)
```

Output:

```text
['api', 'worker', 'database']
```

You can find the number of items with `len()`:

```python
print(len(services))  # 3
```

An empty list is valid:

```python
services = []

print(len(services))  # 0
```

Be careful when accessing an index: an index must exist in the list. For example, accessing `services[0]` when the list is empty raises an `IndexError`.

## 7. Loops: process items one by one

A `for` loop visits each item in an iterable, such as a list or string.

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

Read the loop as:

1. Take the next service from the list.
2. Store that value in the variable `service`.
3. Run the indented block.
4. Repeat until there are no more items.

### Loop through numbers with `range()`

```python
for number in range(3):
    print(number)
```

Output:

```text
0
1
2
```

The stop value is not included. Therefore, `range(3)` produces the values 0, 1, and 2.

### Keep a running total

```python
cpu_samples = [40, 55, 60]

total = 0

for sample in cpu_samples:
    total = total + sample

print(total)  # 155
```

Dry run:

| Step | Current sample | Total before | Total after |
|---:|---:|---:|---:|
| Start | — | — | 0 |
| 1 | 40 | 0 | 40 |
| 2 | 55 | 40 | 95 |
| 3 | 60 | 95 | 155 |

This pattern—visit each item and update a result—is one of the most common foundations of beginner DSA problems.

## 8. Dictionaries: connect a key to a value

A dictionary stores key-value pairs.

```python
service_status = {
    "api": "healthy",
    "worker": "failed",
    "database": "healthy"
}

print(service_status["worker"])  # failed
```

Think of it as a lookup:

- key: `"worker"`
- value: `"failed"`

You can add or update a value by assigning to a key:

```python
service_status["worker"] = "healthy"
service_status["scheduler"] = "healthy"
```

A key lookup using square brackets raises a `KeyError` if the key is absent. You can use `get()` when a missing key should produce a fallback value:

```python
status = service_status.get("unknown-service", "not found")
print(status)  # not found
```

We will study dictionaries in more detail in a later lesson. For now, understand that they are useful when you want to look up a value using a key.

## 9. Sets: keep unique values

A set stores distinct values. Repeated values are represented once.

```python
hostnames = {"server-01", "server-02", "server-01"}

print(hostnames)
print(len(hostnames))  # 2
```

Sets are useful when a problem asks about uniqueness or membership.

```python
known_hosts = {"server-01", "server-02"}

print("server-01" in known_hosts)  # True
print("server-03" in known_hosts)  # False
```

A set does not provide list-style positional indexing. Do not use a set when your task requires preserving a specific sequence of positions.

## 10. Functions: give a task a reusable name

A function groups instructions under a name. Define it with `def`.

```python
def greet():
    print("Welcome to Python DSA")


greet()
```

The indented lines form the function body. The final line calls the function.

### Functions with parameters

A parameter lets the caller provide a value:

```python
def greet_engineer(name):
    print(f"Welcome, {name}")


greet_engineer("Alex")
greet_engineer("Sam")
```

### Functions that return a result

A function can calculate a value and return it to the caller:

```python
def calculate_successful_jobs(total_jobs, failed_jobs):
    return total_jobs - failed_jobs


successful_jobs = calculate_successful_jobs(10, 2)
print(successful_jobs)  # 8
```

Important distinction:

- `print(value)` displays a value.
- `return value` sends a value back from the function.

For DSA exercises, returning a result usually makes a function easier to test and reuse than printing from inside the function.

### A function for finding the first failed service

```python
def find_first_failed_service(services):
    for service in services:
        if service["status"] == "failed":
            return service["name"]

    return None


service_records = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "database", "status": "healthy"}
]

result = find_first_failed_service(service_records)
print(result)  # worker
```

Line-by-line explanation:

1. `def find_first_failed_service(services):` defines a function with one parameter.
2. `for service in services:` visits each record in order.
3. `service["status"] == "failed"` checks the status of the current record.
4. `return service["name"]` immediately returns the first matching service name.
5. `return None` runs if the loop finishes without finding a failed service.

Why return `None`? It communicates that no matching service was found. It is different from returning the text `"None"`.

### Dry run

Input:

```python
services = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "database", "status": "healthy"}
]
```

| Iteration | Current service | Is status failed? | Action |
|---:|---|---|---|
| 1 | api | No | Continue |
| 2 | worker | Yes | Return `"worker"` and stop |

The third record is not examined because the function has already returned.

**Complexity:** In the worst case, the function checks all (n) records, so its time complexity is (O(n)). It uses (O(1)) additional space for the scan itself, excluding the input records.

The example assumes each record contains the keys `"name"` and `"status"`. Input validation for missing or malformed fields is a separate concern.

## 11. Test your functions with `assert`

An assertion checks that an expression is true. It is useful for small learning examples.

```python
def find_first_failed_service(services):
    for service in services:
        if service["status"] == "failed":
            return service["name"]

    return None


sample_services = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"}
]

assert find_first_failed_service(sample_services) == "worker"
assert find_first_failed_service([]) is None
assert find_first_failed_service([
    {"name": "api", "status": "healthy"}
]) is None
assert find_first_failed_service([
    {"name": "worker-1", "status": "failed"},
    {"name": "worker-2", "status": "failed"}
]) == "worker-1"

print("All example checks passed")
```

These checks cover:

- a matching record;
- an empty list;
- no matching record;
- more than one matching record (the first match is returned).

If an assertion condition is false, Python raises an `AssertionError`. Assertions are suitable for learning and internal checks; do not use them as a substitute for validating untrusted production input.

## 12. Common beginner mistakes

| Mistake | Why it causes trouble | Better habit |
|---|---|---|
| Using `=` instead of `==` in a comparison | Assignment and comparison are different operations | Check whether you mean “set” or “compare” |
| Assuming list indexes start at 1 | Python sequence indexes start at 0 | Trace the index and list length |
| Accessing an item in an empty list | No valid index exists | Check whether the collection has an item |
| Printing instead of returning from a function | The caller cannot use the printed text as the function result | Return the computed value |
| Forgetting the indentation of a block | Python uses indentation to group statements | Keep each block consistently indented |
| Assuming a dictionary key exists | A missing key lookup can raise `KeyError` | Validate the key or use `get()` |
| Expecting a set to preserve list positions | Sets are for distinct membership, not positional indexing | Choose the structure that matches the task |
| Changing a collection while looping over it | The iteration can become difficult to reason about | Build a new collection or iterate over a copy when appropriate |

## 13. Practice exercises

Try these before looking at the suggested solutions. Use small inputs first.

### Exercise 1 — Print every hostname

Given:

```python
hostnames = ["web-01", "web-02", "db-01"]
```

Write a loop that prints each hostname on its own line.

### Exercise 2 — Count the servers

Given a list of hostnames, calculate how many entries it contains. Do not manually count them.

### Exercise 3 — Find the first failed job

Given a list of job dictionaries with `"name"` and `"status"` keys, write a function that returns the first failed job's name. Return `None` if there is no failed job.

### Exercise 4 — Calculate a total

Given:

```python
request_counts = [10, 15, 5]
```

Use a loop to calculate the total number of requests.

### Exercise 5 — Check whether a host is known

Given a set of known hostnames, write a condition that checks whether `"web-01"` is present.

### Suggested solutions

<details>
<summary>Exercise 1 — Print every hostname</summary>

```python
hostnames = ["web-01", "web-02", "db-01"]

for hostname in hostnames:
    print(hostname)
```

</details>

<details>
<summary>Exercise 2 — Count the servers</summary>

```python
hostnames = ["web-01", "web-02", "db-01"]

server_count = len(hostnames)
print(server_count)  # 3
```

</details>

<details>
<summary>Exercise 3 — Find the first failed job</summary>

```python
def find_first_failed_job(jobs):
    for job in jobs:
        if job["status"] == "failed":
            return job["name"]

    return None
```

</details>

<details>
<summary>Exercise 4 — Calculate a total</summary>

```python
request_counts = [10, 15, 5]

total = 0

for count in request_counts:
    total += count

print(total)  # 30
```

</details>

<details>
<summary>Exercise 5 — Check whether a host is known</summary>

```python
known_hosts = {"web-01", "web-02", "db-01"}

if "web-01" in known_hosts:
    print("Host is known")
else:
    print("Host is not known")
```

</details>

## 14. Lesson completion checklist

Before moving on, make sure you can:

- [ ] Explain what a variable stores.
- [ ] Recognize strings, integers, floats, and Booleans.
- [ ] Use basic arithmetic and comparison operators.
- [ ] Write a simple `if` / `else` statement.
- [ ] Create a list and access an item by index.
- [ ] Explain why the first list index is zero.
- [ ] Use a `for` loop to process every item.
- [ ] Use a dictionary to look up a value by key.
- [ ] Use a set to check membership and represent unique values.
- [ ] Define a function with a parameter and return a result.
- [ ] Test a function with normal and edge-case examples.
- [ ] Explain the first-failed-service example without reading it line by line.

If one item is unclear, rerun the example and change its input. You do not need to rush to the next topic.

---

## Official Python documentation

These are the authoritative references used for this lesson:

1. [An Informal Introduction to Python](https://docs.python.org/3/tutorial/introduction.html) — expressions, variables, strings, and basic values.
2. [More Control Flow Tools](https://docs.python.org/3/tutorial/controlflow.html) — `if`, `for`, `range()`, and defining functions.
3. [Data Structures](https://docs.python.org/3/tutorial/datastructures.html) — lists, list operations, dictionaries, and sets.
4. [Built-in Functions](https://docs.python.org/3/library/functions.html) — `print()`, `len()`, `type()`, and other built-ins.
5. [Built-in Types](https://docs.python.org/3/library/stdtypes.html) — behavior of built-in numeric, text, and collection types.

The official tutorial is the source for Python language examples. Complexity statements in this lesson describe the shown algorithm and its assumptions; they are algorithm analysis, not a performance guarantee for a particular environment.

---

**Previous:** [DSA for DevOps roadmap](../README.md)

**Next:** [02 — Lists and Iteration](../02%20Lists%20and%20Iteration/README.md) — process a list of services and find matching records.
