# Loops

Loops let Python automation repeat work over collections, streams, and controlled ranges. DevOps scripts use loops to inspect resources, process configuration entries, read command output, retry bounded work, and perform repeated validation.

The goal is to understand `for` and `while`, choose the right loop for the job, control termination explicitly, and avoid accidental infinite or unnecessarily expensive work.

---

## 1. `for` loops

A `for` loop iterates over an iterable.

```python
services = ["api", "worker", "scheduler"]

for service in services:
    print(service)
```

The loop assigns each item to the loop variable and executes the indented block.

For automation, think of a `for` loop as:

```text
source of items
      ↓
process one item
      ↓
next item
      ↓
repeat until exhausted
```

---

## 2. Iterating over strings

Strings are iterable sequences.

```python
name = "api"

for character in name:
    print(character)
```

Be explicit about what represents one unit of work. Iterating over a string gives characters, which may not be what an automation task intends.

---

## 3. Iterating over dictionaries

Iterating over a dictionary directly produces its keys.

```python
config = {
    "environment": "production",
    "region": "ap-south-1",
}

for key in config:
    print(key)
```

Use `.items()` when both keys and values are needed.

```python
for key, value in config.items():
    print(key, value)
```

Use `.values()` when only values are relevant.

```python
for value in config.values():
    print(value)
```

Choose the iteration interface that matches the question being asked.

---

## 4. `range()` for controlled repetition

`range()` produces an iterable representing an arithmetic progression of integers.

```python
for attempt in range(3):
    print(attempt)
```

The stop value is excluded.

```python
for number in range(1, 4):
    print(number)
```

This produces `1`, `2`, and `3`.

Use `range()` when the iteration is naturally based on a sequence of integer values. Do not use it merely because an index happens to exist if direct iteration is clearer.

---

## 5. `enumerate()` instead of manual indexes

When both an item and its position are required, `enumerate()` is usually clearer than maintaining a separate counter.

```python
services = ["api", "worker", "scheduler"]

for index, service in enumerate(services):
    print(index, service)
```

You can provide a starting value:

```python
for number, service in enumerate(services, start=1):
    print(number, service)
```

Prefer this over:

```python
index = 0
for service in services:
    print(index, service)
    index += 1
```

The `enumerate()` form expresses the intent directly.

---

## 6. `zip()` for parallel iteration

`zip()` iterates over multiple iterables together.

```python
services = ["api", "worker"]
ports = [8080, 9090]

for service, port in zip(services, ports):
    print(service, port)
```

The resulting iteration follows the shortest input iterable.

Do not assume two collections have matching lengths unless that assumption is validated or guaranteed by the surrounding design.

---

## 7. Looping over files

Files are iterable, so they can be processed line by line.

```python
with open("services.txt", encoding="utf-8") as file:
    for line in file:
        line = line.strip()
        if line:
            print(line)
```

This pattern avoids requiring the entire file to be represented as a list of lines first.

For operational automation, process data incrementally when the input can be large and the task does not require the entire dataset at once.

---

## 8. `while` loops

A `while` loop repeats while its condition remains truthy.

```python
attempt = 0

while attempt < 3:
    print(attempt)
    attempt += 1
```

A `while` loop is appropriate when termination depends on a condition that changes during the loop rather than simply exhausting an iterable.

---

## 9. Preventing infinite loops

A `while` loop must have a reachable termination condition.

A common safe pattern is:

```python
attempt = 0
max_attempts = 3

while attempt < max_attempts:
    attempt += 1
    print(f"Attempt {attempt}")
```

The loop state changes on every iteration and eventually makes the condition false.

Be especially careful when the loop depends on external state. A service that never becomes healthy, a command that never produces the expected state, or a polling condition that is never satisfied can otherwise keep a process running indefinitely.

---

## 10. `break`

`break` terminates the nearest enclosing loop immediately.

```python
for service in services:
    if service == "worker":
        break
    print(service)
```

This is useful when the required result has been found and continuing would do unnecessary work.

Use `break` when early termination is part of the actual algorithm. Avoid scattering exits through complex loops because that can make control flow difficult to reason about.

---

## 11. `continue`

`continue` skips the remainder of the current iteration and moves to the next iteration.

```python
for service in services:
    if service == "scheduler":
        continue
    print(service)
```

This can be useful for filtering work.

Avoid using many `continue` statements when a straightforward conditional block would make the processing path easier to read.

---

## 12. `else` on loops

Python allows an `else` clause on `for` and `while` loops.

For a loop, the `else` block runs when the loop finishes normally rather than through `break`.

```python
services = ["api", "worker", "scheduler"]

for service in services:
    if service == "database":
        print("Found database")
        break
else:
    print("Database was not found")
```

This behavior can be useful for search operations, but it is unfamiliar to some readers. Use it when it makes the search result clearer; otherwise, an explicit flag or helper function may be easier to maintain.

---

## 13. Nested loops

Loops can be nested.

```python
regions = ["east", "west"]
services = ["api", "worker"]

for region in regions:
    for service in services:
        print(region, service)
```

Nested loops are appropriate when the problem genuinely has multiple dimensions.

However, nested loops can multiply the amount of work. For large datasets, consider whether a different data structure or algorithm can express the task more efficiently.

---

## 14. Looping over a copy when modifying a collection

Do not casually modify a collection while iterating over the same collection.

For example, removing items from a list while iterating can cause elements to be skipped.

Prefer creating a new collection when filtering is the goal:

```python
services = ["api", "worker", "scheduler"]
active_services = [
    service for service in services
    if service != "scheduler"
]
```

If mutation is necessary, make the iteration strategy explicit and test the behavior.

---

## 15. Filtering with conditions

Loops often combine iteration and validation.

```python
services = ["api", "", "worker"]

for service in services:
    if not service:
        continue
    print(service)
```

The important design question is what an empty value means. Do not silently skip invalid data if invalid data should cause the automation to fail.

---

## 16. Accumulating results

A loop can build a result collection.

```python
services = ["api", "worker", "scheduler"]

healthy_services = []

for service in services:
    if service != "scheduler":
        healthy_services.append(service)
```

When the transformation is simple, a list comprehension can be clearer:

```python
healthy_services = [
    service for service in services
    if service != "scheduler"
]
```

Use an ordinary loop when the processing requires multiple statements, exception handling, logging, or other side effects that would make a comprehension difficult to read.

---

## 17. Counting with loops

For simple counting, Python's built-in tools may be clearer than manually maintaining a counter.

For example, if the goal is only to know how many items exist:

```python
count = len(services)
```

If the loop needs a counter tied to each item, `enumerate()` is usually appropriate.

When counting occurrences of values, specialized tools such as `collections.Counter` may be a better fit than a manual dictionary loop.

---

## 18. Searching with a loop

A loop can stop when it finds a required item.

```python
for service in services:
    if service == "api":
        print("API found")
        break
```

For simple membership checks, use the built-in membership operator instead:

```python
if "api" in services:
    print("API found")
```

Use the simplest operation that accurately expresses the requirement.

---

## 19. Polling with `while`

Polling is a common DevOps pattern, but the loop must have explicit termination behavior.

A conceptual bounded polling loop looks like:

```python
attempt = 0
max_attempts = 10

while attempt < max_attempts:
    attempt += 1

    if check_health():
        print("Healthy")
        break
else:
    raise RuntimeError("Health check did not succeed")
```

The exact health-check implementation depends on the system being automated.

A production polling implementation should also define an appropriate delay, timeout strategy, failure behavior, and logging policy rather than spinning continuously.

---

## 20. Retries and loops

Retries are controlled repetition, but a retry loop should not retry every failure indefinitely.

A bounded retry design should make these decisions explicit:

```text
maximum attempts
      ↓
what counts as retryable
      ↓
when to stop
      ↓
what delay/backoff to use
      ↓
what failure to report
```

Do not treat retry logic as a generic solution to every exception. Some failures are permanent, and retrying them can make an incident worse.

This topic teaches loop mechanics. Detailed retry and timeout policy belongs in the repository's dedicated retry topic.

---

## 21. Loop control and exceptions

Exceptions can interrupt normal loop execution.

For example:

```python
for service in services:
    try:
        check_service(service)
    except RuntimeError as exc:
        print(f"Check failed for {service}: {exc}")
```

Whether the loop should continue after an exception is an operational decision.

Do not catch broad exceptions merely to keep the loop moving. Decide which failures are expected, which should stop the operation, and which should be reported.

---

## 22. Avoid unnecessary work inside loops

Operations performed inside a loop may run many times.

Prefer computing invariant values outside the loop when they do not depend on the current item.

Instead of repeatedly constructing the same collection:

```python
allowed = {"dev", "test", "production"}

for environment in environments:
    if environment in allowed:
        process(environment)
```

This also makes the policy easier to identify and test.

For performance-sensitive automation, measure before making complicated optimizations. The simplest correct loop is usually the best starting point.

---

## 23. Loop variables and scope

A `for` loop assigns to its loop variable in the surrounding local scope.

```python
for service in services:
    current = service

print(current)
```

Do not rely on a loop variable after the loop unless that is intentional and its possible empty-input behavior is handled.

For example, if the collection is empty, the loop body never runs and an expected assignment may never happen.

---

## 24. `pass` in loops

`pass` does nothing.

It can be used as a placeholder while developing code:

```python
for service in services:
    pass
```

It does not skip an iteration. Use `continue` when the intended behavior is to move to the next iteration.

---

## 25. Prefer direct iteration

When you need values, iterate over values directly.

Prefer:

```python
for service in services:
    process(service)
```

over:

```python
for index in range(len(services)):
    process(services[index])
```

The direct form communicates the actual intent and avoids unnecessary index management.

Use indexes when the position itself is part of the requirement, or use `enumerate()` when both position and value are needed.

---

## 26. Looping over command output

When processing output from a command, keep parsing separate from action when possible.

Conceptually:

```python
for line in output_lines:
    line = line.strip()

    if not line:
        continue

    if line.startswith("ERROR"):
        report_failure(line)
```

For structured command output, prefer a structured output format and parser when the command supports one. Do not build fragile parsing rules around incidental human-readable formatting when a stable machine-readable interface exists.

---

## 27. Loops and idempotency

A loop can apply an operation to many resources, so the operation should be designed carefully.

For example:

```text
for each resource
       ↓
inspect current state
       ↓
compare with desired state
       ↓
change only when necessary
```

This is a repository design recommendation for safe automation, not a property automatically provided by Python loops.

A loop itself does not make an operation idempotent.

---

## 28. Common mistakes

### Mistake 1: Infinite `while` loops

Ensure the loop has a reachable termination condition.

### Mistake 2: Modifying a collection while iterating over it

Prefer a new filtered collection or an explicit safe strategy.

### Mistake 3: Using indexes unnecessarily

Prefer direct iteration or `enumerate()`.

### Mistake 4: Retrying forever

Bound retries and define failure behavior.

### Mistake 5: Swallowing exceptions in loops

Continuing after an error should be intentional and observable.

### Mistake 6: Parsing fragile human-readable output

Prefer stable structured interfaces when available.

### Mistake 7: Doing expensive invariant work repeatedly

Move loop-independent work outside the loop when appropriate.

### Mistake 8: Using `break` or `continue` without clear intent

Control-flow shortcuts should make the algorithm clearer, not harder to follow.

### Mistake 9: Assuming the loop ran at least once

Empty input means a `for` loop may execute zero times.

### Mistake 10: Treating loops as a substitute for a better data structure

Sometimes a set, dictionary, `Counter`, or other standard-library tool expresses the problem more directly.

---

## 29. Practical DevOps exercises

### Exercise 1 — Service inventory

Iterate over a list of service names and print each service with its position using `enumerate()`.

### Exercise 2 — Configuration validation

Iterate over configuration entries and report which entries are missing required values.

### Exercise 3 — Deployment filtering

Given a collection of services, process only services belonging to an allowed set.

### Exercise 4 — Bounded polling

Implement a loop that checks a condition a fixed number of times and reports failure when the condition never becomes true.

### Exercise 5 — Search

Find the first unhealthy service and stop processing once it is found.

### Exercise 6 — Parallel collections

Use `zip()` to associate service names with their configured ports and validate each pair.

### Exercise 7 — File processing

Read a text file line by line and report lines that match a simple operational rule.

### Exercise 8 — Safe filtering

Create a new collection containing only valid resources instead of deleting items from the collection being iterated.

### Exercise 9 — Retry policy

Design a bounded retry loop that distinguishes retryable failures from permanent failures.

### Exercise 10 — Idempotent action planning

For each resource, compare current and desired state before deciding whether an action is necessary.

---

## 30. Knowledge check

You should be able to explain:

1. What a `for` loop iterates over.
2. When `for` is preferable to `while`.
3. How `range()` behaves with its stop value.
4. Why `enumerate()` is useful.
5. How `zip()` behaves when input lengths differ.
6. What `break` and `continue` do.
7. When a loop `else` block runs.
8. Why modifying a collection during iteration can be dangerous.
9. How to design a bounded polling loop.
10. Why retry loops need explicit termination and failure policy.
11. Why direct iteration is often clearer than indexing.
12. Why loop mechanics alone do not guarantee idempotent automation.

---

## 31. Completion criteria

You have completed this topic when you can:

- iterate over lists, dictionaries, strings, and files;
- use `range()` appropriately;
- use `enumerate()` when positions matter;
- use `zip()` for coordinated iteration;
- choose between `for` and `while` based on the termination model;
- use `break` and `continue` intentionally;
- reason about loop `else` behavior;
- write bounded polling and retry loops;
- avoid unsafe collection mutation during iteration;
- separate parsing, validation, and operational actions;
- keep repeated automation observable and safely bounded.

---

## Verified facts vs repository design

### Verified facts

The Python documentation defines `for` statements, `while` statements, loop control statements such as `break` and `continue`, the `else` clause on loops, `range()`, `enumerate()`, and `zip()` behavior.

### Repository design decisions

This topic emphasizes bounded polling, explicit retry limits, operational validation, idempotency, and separating parsing from action because the repository targets DevOps automation. These are engineering recommendations, not requirements of Python's loop syntax.

### Demonstrations

The service, deployment, health-check, configuration, and resource examples are intentionally generic demonstrations. They do not claim behavior for a particular cloud provider, Kubernetes implementation, CI/CD platform, or service manager.

---

## Authoritative references

- Python Tutorial — More Control Flow Tools: https://docs.python.org/3/tutorial/controlflow.html
- Python Tutorial — `for` Statements: https://docs.python.org/3/tutorial/controlflow.html#for-statements
- Python Tutorial — `break` and `continue`: https://docs.python.org/3/tutorial/controlflow.html#break-and-continue-statements-and-else-clauses-on-loops
- Python Built-in Functions — `range()`: https://docs.python.org/3/library/functions.html#func-range
- Python Built-in Functions — `enumerate()`: https://docs.python.org/3/library/functions.html#enumerate
- Python Built-in Functions — `zip()`: https://docs.python.org/3/library/functions.html#zip
- Python Language Reference — Compound statements: https://docs.python.org/3/reference/compound_stmts.html
