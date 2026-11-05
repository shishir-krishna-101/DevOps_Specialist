# 04 — Counting with Loops

Many small automation tasks need a summary of repeated events:

- How many jobs succeeded?
- How many jobs failed?
- How many services are healthy?
- How many records have a warning status?
- How many times did each result appear in a sample?

This lesson teaches you to count items by visiting a collection with a loop. You will start with one counter, then count multiple categories with a dictionary.

> **Repository design decision:** We use small invented job and service records to teach counting. These examples do not define how any CI/CD platform, cloud service, or monitoring product reports its results.

---

## 1. What does counting mean?

Suppose a list contains the result of each job:

```python
job_results = ["success", "failed", "success", "success", "failed"]
```

We want to answer:

- How many jobs succeeded?
- How many jobs failed?
- How many results are in the list?

The list has five items. Three are `"success"` and two are `"failed"`.

| Result | Count |
|---|---:|
| success | 3 |
| failed | 2 |
| Total | 5 |

A **counter** is a variable that stores a number and is updated when an item meets a condition.

## 2. Count all items with a loop

First, count how many items are in a list without using `len()`. This helps show the basic pattern.

```python
job_results = ["success", "failed", "success", "success", "failed"]

total = 0

for result in job_results:
    total = total + 1

print(total)
```

Output:

```text
5
```

### Understand each line

- `total = 0`: start the counter at zero.
- `for result in job_results:`: visit each item in order.
- `total = total + 1`: add one for every item.
- `print(total)`: display the final count after the loop.

The shorter update form is:

```python
total += 1
```

It means the same as `total = total + 1`.

### Dry run

| Item visited | Counter before | Counter after |
|---|---:|---:|
| success | 0 | 1 |
| failed | 1 | 2 |
| success | 2 | 3 |
| success | 3 | 4 |
| failed | 4 | 5 |

The loop executes once per item, so it performs O(n) work for a list of n items. The counter uses O(1) additional space.

Python also provides `len(job_results)` for the length of a collection. We use the loop here to learn the counting pattern.

## 3. Count successful jobs

Now increase the counter only when the current item is `"success"`.

```python
job_results = ["success", "failed", "success", "success", "failed"]

success_count = 0

for result in job_results:
    if result == "success":
        success_count += 1

print(success_count)
```

Output:

```text
3
```

### Line-by-line explanation

1. Create the input list.
2. Set `success_count` to zero.
3. Visit every result.
4. Compare the current result with the exact string `"success"`.
5. Increase the counter only when the comparison is true.
6. Print the counter after all items have been visited.

### Dry run

| Current result | Is it success? | success_count after check |
|---|---|---:|
| success | Yes | 1 |
| failed | No | 1 |
| success | Yes | 2 |
| success | Yes | 3 |
| failed | No | 3 |

Notice that the counter does not increase for a failed result.

## 4. Count failed jobs

The same loop pattern can count failures. Only the condition changes.

```python
job_results = ["success", "failed", "success", "success", "failed"]

failed_count = 0

for result in job_results:
    if result == "failed":
        failed_count += 1

print(failed_count)
```

Output:

```text
2
```

### Count two categories in one loop

You do not need to traverse the list separately for every category. Use two counters:

```python
job_results = ["success", "failed", "success", "success", "failed"]

success_count = 0
failed_count = 0

for result in job_results:
    if result == "success":
        success_count += 1
    elif result == "failed":
        failed_count += 1

print("Success:", success_count)
print("Failed:", failed_count)
```

Output:

```text
Success: 3
Failed: 2
```

The `elif` branch is checked only when the first condition was false. A result that is neither `"success"` nor `"failed"` does not increase either counter.

### What about other statuses?

A real dataset might include values such as `"skipped"`, `"cancelled"`, or `"running"`. Do not silently classify every unknown value as failed. Decide what categories the task requires and how unknown values should be handled.

For example, you can count unrecognized values separately:

```python
job_results = ["success", "failed", "skipped", "success", "cancelled"]

success_count = 0
failed_count = 0
other_count = 0

for result in job_results:
    if result == "success":
        success_count += 1
    elif result == "failed":
        failed_count += 1
    else:
        other_count += 1

print("Success:", success_count)
print("Failed:", failed_count)
print("Other:", other_count)
```

Output:

```text
Success: 2
Failed: 1
Other: 2
```

The `else` branch means “anything not matched by the earlier conditions.” It does not mean “failed.”

## 5. Count status values with a dictionary

Two counters work well when you have only a few known categories. What if the input can contain many different status values?

A dictionary can store a count for each status.

```python
job_results = [
    "success",
    "failed",
    "success",
    "skipped",
    "failed",
    "success"
]

counts = {}

for result in job_results:
    if result in counts:
        counts[result] += 1
    else:
        counts[result] = 1

print(counts)
```

Output:

```text
{'success': 3, 'failed': 2, 'skipped': 1}
```

### How the dictionary is used

A dictionary stores key-value pairs. In this example:

- The key is the status text.
- The value is the number of times that status has appeared.

The loop follows this rule:

1. If the status already exists as a key, add one to its count.
2. Otherwise, create the key and set its first count to one.

### Dry run

Input: `["success", "failed", "success", "skipped", "failed"]`

| Current result | Dictionary after processing |
|---|---|
| success | `{"success": 1}` |
| failed | `{"success": 1, "failed": 1}` |
| success | `{"success": 2, "failed": 1}` |
| skipped | `{"success": 2, "failed": 1, "skipped": 1}` |
| failed | `{"success": 2, "failed": 2, "skipped": 1}` |

The final dictionary is the frequency count.

### A shorter version using `get()`

The dictionary method `get(key, default)` returns the value for a key, or the supplied default when the key is absent.

```python
counts = {}

for result in job_results:
    counts[result] = counts.get(result, 0) + 1
```

Read the update in small steps:

- `counts.get(result, 0)` gets the current count, or zero if this status has not appeared.
- `+ 1` adds the current occurrence.
- `counts[result] = ...` stores the updated count.

Both versions produce the same counts. The longer version may be easier to understand when you are first learning dictionaries.

## 6. Count healthy and unhealthy services

Now use a list of dictionaries, similar to a small service inventory.

```python
services = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "database", "status": "healthy"},
    {"name": "scheduler", "status": "degraded"}
]

healthy_count = 0
unhealthy_count = 0

for service in services:
    if service["status"] == "healthy":
        healthy_count += 1
    else:
        unhealthy_count += 1

print("Healthy:", healthy_count)
print("Not healthy:", unhealthy_count)
```

Output:

```text
Healthy: 2
Not healthy: 2
```

In this exercise, “not healthy” means every record whose status is not exactly `"healthy"`. That includes both `"failed"` and `"degraded"` in the sample.

If you need to distinguish degraded from failed, use separate categories instead of grouping them together.

### Count every service status

```python
services = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "database", "status": "healthy"},
    {"name": "scheduler", "status": "degraded"}
]

status_counts = {}

for service in services:
    status = service["status"]
    status_counts[status] = status_counts.get(status, 0) + 1

print(status_counts)
```

Output:

```text
{'healthy': 2, 'failed': 1, 'degraded': 1}
```

This example assumes every record has a `"status"` key containing a string. Validating malformed or incomplete records is an additional task.

## 7. Count values that meet a condition

Counting is not limited to exact text matches. You can count numeric values that satisfy a condition.

Example: count sample CPU measurements at or above a teaching threshold of 80.

```python
cpu_samples = [35, 82, 91, 67, 80, 45]

threshold = 80
high_cpu_count = 0

for cpu in cpu_samples:
    if cpu >= threshold:
        high_cpu_count += 1

print(high_cpu_count)
```

Output:

```text
3
```

The matching values are `82`, `91`, and `80`.

The threshold is invented for this exercise. It is not a universal alerting threshold or production recommendation.

### Dry run

| CPU sample | Is it at least 80? | Counter |
|---:|---|---:|
| 35 | No | 0 |
| 82 | Yes | 1 |
| 91 | Yes | 2 |
| 67 | No | 2 |
| 80 | Yes | 3 |
| 45 | No | 3 |

## 8. Calculate a percentage safely

A percentage can help describe a count relative to the total.

For this example, calculate the percentage of successful jobs:

```python
job_results = ["success", "failed", "success", "success", "failed"]

success_count = 0

for result in job_results:
    if result == "success":
        success_count += 1

total_count = len(job_results)

if total_count > 0:
    success_percentage = (success_count / total_count) * 100
    print(success_percentage)
else:
    print("No job results available")
```

Output:

```text
60.0
```

Formula:

`(successful jobs / total jobs) × 100`

### Why check the total?

If the list is empty, the total is zero. Dividing by zero is invalid, so the program checks the total before calculating the percentage.

Do not report an empty dataset as 0% success without deciding that this is the intended meaning. “No data” and “zero successful jobs out of some jobs” are different situations.

## 9. Put the counting logic in a function

Functions make the counting logic reusable.

```python
def count_results(results):
    counts = {}

    for result in results:
        counts[result] = counts.get(result, 0) + 1

    return counts


sample_results = ["success", "failed", "success", "skipped"]

summary = count_results(sample_results)

print(summary)
```

Output:

```text
{'success': 2, 'failed': 1, 'skipped': 1}
```

### Line-by-line explanation

- `def count_results(results):` defines a function that accepts a collection of results.
- `counts = {}` creates an empty dictionary for the summary.
- `for result in results:` visits each result.
- `counts.get(result, 0)` reads the existing count, or zero for a new status.
- `+ 1` adds one for the current occurrence.
- `counts[result] = ...` saves the updated count.
- `return counts` gives the completed dictionary back to the caller.

### Complexity

Let:

- `n` = number of input results;
- `k` = number of distinct result values.

For this dictionary-counting algorithm:

- **Time:** O(n) expected overall under the usual hash-dictionary operation model, because the loop processes each input item and updates one dictionary entry.
- **Additional space:** O(k), because the dictionary stores one entry per distinct value.

The dictionary can contain at most one entry per distinct status in the input.

## 10. Common beginner mistakes

| Mistake | Why it is a problem | Better habit |
|---|---|---|
| Starting a counter at 1 | The result is one too high before any item is counted | Start at 0 |
| Incrementing outside the condition | Every item gets counted, not just matching items | Indent the increment inside the matching branch |
| Using `=` instead of `==` in a condition | Assignment and comparison are different operations | Use `==` to compare values |
| Forgetting a new dictionary key | Updating a missing key with `counts[key] += 1` raises `KeyError` | Check membership or use `get(key, 0)` |
| Treating every unknown status as failed | Other categories may exist | Define the categories the task requires |
| Dividing by zero | An empty input has no total to divide by | Check that the total is greater than zero |
| Counting only one item | Returning or breaking too early stops the loop | Let the loop continue when the task needs a full count |
| Assuming sample statuses are exhaustive | Real inputs may contain additional values | Validate or explicitly account for unexpected categories |

## 11. Practice exercises

Try to solve each exercise before opening its suggested solution.

### Exercise 1 — Count all jobs

Given a list of job results, count the total number of items using a loop.

### Exercise 2 — Count successes

Count how many times `"success"` appears in a list.

### Exercise 3 — Count failures

Count how many times `"failed"` appears.

### Exercise 4 — Count two categories

Count success and failure results in one loop. Do not include other statuses in either count.

### Exercise 5 — Count every status

Use a dictionary to count each distinct status in the input list.

### Exercise 6 — Count unhealthy services

Given service dictionaries, count records whose status is not `"healthy"`.

### Exercise 7 — Count high CPU samples

Given a list of CPU samples and a threshold, count values greater than or equal to the threshold.

### Exercise 8 — Calculate success percentage

Count successful jobs and calculate the percentage. Handle an empty list without dividing by zero.

---

## Suggested solutions

<details>
<summary>Exercise 1 — Count all jobs</summary>

```python
def count_all(items):
    total = 0

    for item in items:
        total += 1

    return total


assert count_all(["success", "failed", "success"]) == 3
assert count_all(["success"]) == 1
assert count_all([]) == 0
```

</details>

<details>
<summary>Exercise 2 — Count successes</summary>

```python
def count_successes(results):
    count = 0

    for result in results:
        if result == "success":
            count += 1

    return count


assert count_successes(["success", "failed", "success"]) == 2
assert count_successes(["failed"]) == 0
assert count_successes([]) == 0
```

</details>

<details>
<summary>Exercise 3 — Count failures</summary>

```python
def count_failures(results):
    count = 0

    for result in results:
        if result == "failed":
            count += 1

    return count


assert count_failures(["success", "failed", "failed"]) == 2
assert count_failures(["success"]) == 0
assert count_failures([]) == 0
```

</details>

<details>
<summary>Exercise 4 — Count two categories</summary>

```python
def count_success_and_failure(results):
    success_count = 0
    failed_count = 0

    for result in results:
        if result == "success":
            success_count += 1
        elif result == "failed":
            failed_count += 1

    return success_count, failed_count


assert count_success_and_failure(
    ["success", "failed", "skipped", "success"]
) == (2, 1)

assert count_success_and_failure([]) == (0, 0)
```

</details>

<details>
<summary>Exercise 5 — Count every status</summary>

```python
def count_statuses(results):
    counts = {}

    for result in results:
        counts[result] = counts.get(result, 0) + 1

    return counts


assert count_statuses(
    ["success", "failed", "success", "skipped"]
) == {"success": 2, "failed": 1, "skipped": 1}

assert count_statuses([]) == {}
```

</details>

<details>
<summary>Exercise 6 — Count unhealthy services</summary>

```python
def count_unhealthy(services):
    count = 0

    for service in services:
        if service["status"] != "healthy":
            count += 1

    return count


sample = [
    {"name": "api", "status": "healthy"},
    {"name": "worker", "status": "failed"},
    {"name": "scheduler", "status": "degraded"}
]

assert count_unhealthy(sample) == 2
assert count_unhealthy([]) == 0
```

</details>

<details>
<summary>Exercise 7 — Count high CPU samples</summary>

```python
def count_at_or_above(samples, threshold):
    count = 0

    for value in samples:
        if value >= threshold:
            count += 1

    return count


assert count_at_or_above([35, 82, 91, 67, 80], 80) == 3
assert count_at_or_above([10, 20], 80) == 0
assert count_at_or_above([], 80) == 0
```

</details>

<details>
<summary>Exercise 8 — Calculate success percentage</summary>

```python
def success_percentage(results):
    if not results:
        return None

    success_count = 0

    for result in results:
        if result == "success":
            success_count += 1

    return (success_count / len(results)) * 100


assert success_percentage(
    ["success", "failed", "success", "success", "failed"]
) == 60.0

assert success_percentage(["failed", "failed"]) == 0.0
assert success_percentage([]) is None
```

</details>

## 12. Test the main counting function

Use assertions to verify normal input, repeated values, and empty input.

```python
def count_results(results):
    counts = {}

    for result in results:
        counts[result] = counts.get(result, 0) + 1

    return counts


assert count_results(
    ["success", "failed", "success", "skipped", "failed"]
) == {"success": 2, "failed": 2, "skipped": 1}

assert count_results(["success"]) == {"success": 1}
assert count_results(["failed", "failed"]) == {"failed": 2}
assert count_results([]) == {}

print("All checks passed")
```

Expected output:

```text
All checks passed
```

For your own tests, include:

- an empty list;
- one item;
- repeated identical values;
- all values different;
- a status not included in your main categories;
- a list containing only failures;
- a list containing only successes.

## 13. Lesson completion checklist

- [ ] Explain what a counter variable does.
- [ ] Count every item in a list with a loop.
- [ ] Count items that match a condition.
- [ ] Count two known categories in one pass.
- [ ] Explain why an `else` branch can represent other statuses.
- [ ] Build a dictionary of status frequencies.
- [ ] Explain `dict.get(key, 0)`.
- [ ] Count service records by their status field.
- [ ] Count numeric values that meet a threshold.
- [ ] Calculate a percentage while handling empty input.
- [ ] Explain O(n) time for one pass through the input.
- [ ] Explain O(k) additional dictionary space for k distinct values.
- [ ] Test empty input and unexpected categories.

---

## Official Python documentation

1. [The Python Tutorial — for statements](https://docs.python.org/3/tutorial/controlflow.html#for-statements) — explains how Python's `for` statement iterates over items in a sequence.
2. [The Python Tutorial — if statements](https://docs.python.org/3/tutorial/controlflow.html#if-statements) — documents conditional branching.
3. [Data Structures — Dictionaries](https://docs.python.org/3/tutorial/datastructures.html#dictionaries) — explains dictionary key-value storage and dictionary operations.
4. [Built-in Types — Mapping Types: dict](https://docs.python.org/3/library/stdtypes.html#mapping-types-dict) — documents dictionary behavior and methods, including `get()`.

The Python documentation supports the language and dictionary behavior described here. Complexity statements describe the counting algorithms in this lesson and state their assumptions.

---

**Previous:** [03 — Minimum, Maximum, and Search](../03%20Minimum%20Maximum%20and%20Search/README.md)

**Next:** [05 — Sets and Duplicates](../05%20Sets%20and%20Duplicates/README.md) — find repeated hostnames and understand how sets store unique values.
