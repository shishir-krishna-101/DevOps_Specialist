# Queues in Python

A queue is a data structure that processes items in the order they arrive.

This lesson introduces the **First In, First Out (FIFO)** pattern using Python's `collections.deque`. The examples use small DevOps-style job and event records to make the idea practical.

> **Learning goal:** Understand how to add work to the end of a queue and process the oldest item from the front.

## 1. What is a queue?

**FIFO** means **First In, First Out**:

- The first item added is the first item removed.
- New items join at the back.
- Processing takes place at the front.

Think of a line of deployment jobs waiting to be processed. If job A arrives before job B, a simple FIFO queue processes A before B.

| Operation | Meaning |
| --- | --- |
| Enqueue | Add an item to the back |
| Dequeue | Remove the item at the front |
| Front / peek | Inspect the next item without removing it |

A queue is different from a stack:

| Data structure | Rule | Example |
| --- | --- | --- |
| Queue | FIFO | Process jobs in arrival order |
| Stack | LIFO | Return to the most recently added item |

These are data-structure examples, not claims about how a particular production scheduler or deployment platform works.

## 2. Why might a DevOps engineer learn queues?

Queue concepts help when reasoning about work that waits to be processed:

- A list of automation tasks waiting for a worker.
- Events waiting for a simple processing loop.
- Files waiting to be handled in arrival order.
- A sequence of health checks that should be examined one at a time.

Real systems may use message brokers, distributed queues, or platform-specific scheduling mechanisms. The in-memory Python examples here are deliberately small learning demonstrations; they are not production job queues.

## 3. Create a queue with `collections.deque`

Python's official tutorial recommends `collections.deque` for queues. It is designed for fast appends and pops from either end.

```python
from collections import deque

jobs = deque()

jobs.append("build")
jobs.append("test")
jobs.append("deploy")

print(jobs)
```

Output:

```text
deque(['build', 'test', 'deploy'])
```

The `append()` method adds an item to the right-hand end of the deque.

### Remove the oldest item

Use `popleft()` to remove an item from the left-hand end:

```python
from collections import deque

jobs = deque(["build", "test", "deploy"])

next_job = jobs.popleft()

print(next_job)
print(jobs)
```

Output:

```text
build
deque(['test', 'deploy'])
```

The `build` job was first in, so it was first out.

### Inspect the next item without removing it

A deque supports indexing. For this simple example, index `0` is the next item:

```python
from collections import deque

jobs = deque(["build", "test", "deploy"])

if jobs:
    print("Next job:", jobs[0])
    print("Queue still contains:", jobs)
```

Output:

```text
Next job: build
Queue still contains: deque(['build', 'test', 'deploy'])
```

Checking that the queue is not empty before reading its first item avoids an invalid index access.

## 4. Why not use a list with `pop(0)`?

A list can hold items in arrival order, but removing its first item requires the remaining items to shift.

```python
jobs = ["build", "test", "deploy"]

next_job = jobs.pop(0)

print(next_job)
print(jobs)
```

This works, but Python's tutorial explains that removing from the beginning of a list is slow because the other elements have to be shifted by one. For a queue that repeatedly removes its oldest item, prefer `deque.popleft()`.

Official explanation: [Using Lists as Queues](https://docs.python.org/3/tutorial/datastructures.html#using-lists-as-queues).

## 5. DevOps-style example: process jobs in arrival order

This is a local Python demonstration. It does not start real builds or deployments.

```python
from collections import deque

pending_jobs = deque([
    "build-api",
    "run-tests",
    "package-image",
])

while pending_jobs:
    job = pending_jobs.popleft()
    print(f"Processing: {job}")

print("All demonstration jobs processed.")
```

Output:

```text
Processing: build-api
Processing: run-tests
Processing: package-image
All demonstration jobs processed.
```

### Line-by-line explanation

1. Import `deque` from Python's `collections` module.
2. Create a queue containing three job names in arrival order.
3. `while pending_jobs:` continues while at least one item remains.
4. `popleft()` removes the oldest job.
5. Print the job that the demonstration is processing.
6. When the queue becomes empty, the loop stops.

### Dry run

| Step | Queue before processing | Removed item | Queue after processing |
| --- | --- | --- | --- |
| 1 | build-api, run-tests, package-image | build-api | run-tests, package-image |
| 2 | run-tests, package-image | run-tests | package-image |
| 3 | package-image | package-image | empty |

The order is preserved: the first job added is the first job processed.

## 6. Handle an empty queue safely

Calling `popleft()` on an empty deque raises `IndexError`. Check the queue before removing an item:

```python
from collections import deque

jobs = deque()

if jobs:
    job = jobs.popleft()
    print("Processing:", job)
else:
    print("No jobs are waiting.")
```

Output:

```text
No jobs are waiting.
```

The `while jobs:` pattern in the previous example also prevents calling `popleft()` after the queue becomes empty.

## 7. Build a reusable queue-processing function

```python
from collections import deque


def process_jobs(job_names):
    pending_jobs = deque(job_names)
    processed_jobs = []

    while pending_jobs:
        job = pending_jobs.popleft()
        processed_jobs.append(job)

    return processed_jobs


jobs = ["backup", "health-check", "cleanup"]

result = process_jobs(jobs)

print(result)
```

Output:

```text
['backup', 'health-check', 'cleanup']
```

### Understand the function

- `job_names` is the input collection.
- `deque(job_names)` creates a queue containing those items.
- `processed_jobs` stores the items removed from the queue.
- The loop removes items from the front until the queue is empty.
- The function returns the processing order.

The function models order only. It does not execute the named operations.

## 8. Basic tests

You can use assertions to check the behavior:

```python
from collections import deque


jobs = deque()
jobs.append("first")
jobs.append("second")

assert jobs[0] == "first"
assert jobs.popleft() == "first"
assert jobs.popleft() == "second"
assert not jobs

print("Queue tests passed.")
```

Expected output:

```text
Queue tests passed.
```

Try changing the order of the two `append()` calls. The first item appended should still be the first one returned by `popleft()`.

## 9. Time and space complexity

Let `n` be the number of items in the queue.

| Operation | Typical complexity | Explanation |
| --- | --- | --- |
| `append(item)` | O(1) | Add one item at the right end |
| `popleft()` | O(1) | Remove one item from the left end |
| Process all `n` items | O(n) | Each item is removed once |
| Store `n` queued items | O(n) | The queue holds the items |

The Python documentation describes deque as designed for fast appends and pops at both ends. The complexity labels above are the conventional algorithm-analysis model for these endpoint operations, rather than a formal worst-case complexity guarantee stated by the tutorial.

## 10. When should you use `queue.Queue`?

For the simple single-threaded FIFO exercises in this lesson, `collections.deque` is a good fit.

Python also provides the `queue` module. Its documentation describes synchronized queue classes intended for use in threaded programming when information must be exchanged safely between multiple threads.

Do not treat `deque` and `queue.Queue` as interchangeable in every situation. Choose based on the coordination requirements of the program.

Official reference: [`queue` — A synchronized queue class](https://docs.python.org/3/library/queue.html).

## 11. Common beginner mistakes

1. **Using `pop()` instead of `popleft()`:** `pop()` removes from the right, which behaves like a stack.
2. **Removing from an empty queue:** Check `if queue` or use `while queue` before calling `popleft()`.
3. **Using `list.pop(0)` repeatedly:** The remaining list elements must shift.
4. **Confusing queue order with priority:** A basic FIFO queue processes arrival order; it does not automatically choose the most important job.
5. **Assuming this is a production scheduler:** The examples only demonstrate an in-memory data structure.

## 12. Beginner exercises

Try each exercise before opening its solution.

### Exercise 1: Create a queue

Create a deque with three server names and print it.

<details>
<summary>Solution</summary>

```python
from collections import deque

servers = deque(["web-01", "web-02", "worker-01"])
print(servers)
```

</details>

### Exercise 2: Add a job

Start with an empty deque and add `"run-tests"` and `"build-image"` in that order.

<details>
<summary>Solution</summary>

```python
from collections import deque

jobs = deque()
jobs.append("run-tests")
jobs.append("build-image")

print(jobs)
```

</details>

### Exercise 3: Remove the oldest item

Given `deque(["backup", "scan", "cleanup"])`, remove and print the first item.

<details>
<summary>Solution</summary>

```python
from collections import deque

jobs = deque(["backup", "scan", "cleanup"])
print(jobs.popleft())
```

</details>

### Exercise 4: Process all jobs

Use a `while` loop to remove and print every item from a queue.

<details>
<summary>Solution</summary>

```python
from collections import deque

jobs = deque(["job-a", "job-b", "job-c"])

while jobs:
    print(jobs.popleft())
```

</details>

### Exercise 5: Handle no work

Print `"Queue is empty"` when a deque has no items. Otherwise, print and remove its first item.

<details>
<summary>Solution</summary>

```python
from collections import deque

jobs = deque()

if jobs:
    print(jobs.popleft())
else:
    print("Queue is empty")
```

</details>

### Exercise 6: Preserve arrival order

Write a function that accepts a list of task names, processes them with a deque, and returns the names in processing order.

<details>
<summary>Solution</summary>

```python
from collections import deque


def processing_order(tasks):
    pending = deque(tasks)
    result = []

    while pending:
        result.append(pending.popleft())

    return result


assert processing_order(["a", "b", "c"]) == ["a", "b", "c"]
```

</details>

## 13. Interview questions

1. What does FIFO stand for?
2. What is the difference between a queue and a stack?
3. What is the difference between `append()` and `popleft()` on a deque?
4. Why is `list.pop(0)` inefficient for repeated queue operations?
5. What happens if you call `popleft()` on an empty deque?
6. When might Python's `queue.Queue` be more suitable than a plain deque?
7. What is the time complexity of processing all items in a queue once?

## 14. Completion checklist

- [ ] I can explain FIFO in my own words.
- [ ] I can create a deque.
- [ ] I can add an item with `append()`.
- [ ] I can remove the oldest item with `popleft()`.
- [ ] I can safely handle an empty queue.
- [ ] I can explain why repeated `list.pop(0)` is not the preferred queue implementation.
- [ ] I can trace a queue by hand.
- [ ] I can explain when a synchronized queue may be needed.

## 15. Official references

Use the Python documentation as the source of truth for the language and library behavior:

- [Python Tutorial — Using Lists as Queues](https://docs.python.org/3/tutorial/datastructures.html#using-lists-as-queues)
- [Python Standard Library — `collections.deque`](https://docs.python.org/3/library/collections.html#collections.deque)
- [Python Standard Library — `queue`](https://docs.python.org/3/library/queue.html)

## Navigation

- Previous: [10 Stacks](../10%20Stacks/README.md)
- Also study: [09 Searching](../09%20Searching/README.md)
- Next: [12 Sliding Window Basics](../12%20Sliding%20Window%20Basics/README.md)
- [DSA for DevOps with Python — Roadmap](../README.md)

---

**Next step:** Practice implementing FIFO with `deque` before moving on to sliding-window problems.
