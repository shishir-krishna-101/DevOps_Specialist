# Debugging and Optimization

> Diagnose incorrect behavior, prove that a fix works, and improve performance without changing the required behavior.

This guide is part of **DSA Interview Preparation**. It focuses on Python coding interviews and practical debugging relevant to DevOps, SRE, platform engineering, and automation.

## Table of Contents

1. [Learning objectives](#learning-objectives)
2. [A repeatable debugging workflow](#a-repeatable-debugging-workflow)
3. [Python debugging tools](#python-debugging-tools)
4. [Testing a suspected bug](#testing-a-suspected-bug)
5. [Common algorithmic bugs](#common-algorithmic-bugs)
6. [A disciplined optimization workflow](#a-disciplined-optimization-workflow)
7. [Measuring runtime and memory](#measuring-runtime-and-memory)
8. [DevOps and SRE debugging scenarios](#devops-and-sre-debugging-scenarios)
9. [Hands-on debugging exercises](#hands-on-debugging-exercises)
10. [Interview communication framework](#interview-communication-framework)
11. [Readiness checklist](#readiness-checklist)
12. [References](#references)

---

## Learning objectives

You should be able to:

- Reproduce a failure and reduce it to a small input.
- Read a traceback and identify the relevant failing operation.
- Use breakpoints, stepping, variable inspection, and logging.
- Turn a reported failure into a regression test.
- Check boundaries, invariants, mutation, ordering, and state transitions.
- Distinguish theoretical complexity from measured runtime.
- Use standard-library tools to measure execution and Python memory allocations.
- Optimize based on evidence while preserving correctness.

Passing a few examples does not prove correctness for all valid inputs. Tests provide evidence; clear contracts, invariants, reasoning, and suitable coverage provide stronger confidence.

## A repeatable debugging workflow

### Step 1 — Understand the contract

Write down valid inputs, expected output, whether empty inputs and duplicates are allowed, whether ordering matters, whether mutation is allowed, and what should happen for invalid input. For time-based logic, define whether interval boundaries are inclusive or exclusive.

Many bugs are contract misunderstandings rather than difficult algorithm errors.

### Step 2 — Reproduce the failure

Find the smallest input that demonstrates the incorrect result. For a function that returns the first index whose value is at least a target in a sorted list, test an empty list, a target below all values, a target above all values, equality at either end, and duplicate target values.

### Step 3 — Trace the state

Inspect loop indices, pointers, window boundaries, running totals, queue or stack contents, heap contents, visited nodes, recursion arguments, and cached state. Ask what each variable means. If its meaning is unclear, the algorithm may lack a clear invariant.

### Step 4 — State the invariant

An invariant is a condition that should remain true at a defined point in an algorithm.

Examples:
- In binary search, the candidate answer remains inside the active search interval.
- In a sliding window, the window represents exactly the elements included in the calculation.
- In BFS, a node is marked visited according to one consistent discovery rule.
- In a min-heap scheduler, the root represents the smallest pending priority.

Check that the invariant is true before the loop, preserved by each iteration, and sufficient when the loop ends.

### Step 5 — Fix the cause, not the symptom

A robust fix explains why the incorrect state was possible and why the new logic prevents it. Avoid unexplained special cases, silently catching every exception, reordering data merely to pass one example, or changing expected output to match incorrect behavior.

### Step 6 — Add a regression test

Keep the smallest failing input as a permanent test. Add nearby boundary cases so the same class of bug is less likely to return.

### Step 7 — Re-run relevant tests

Re-run the original failure, boundary and empty-input cases, duplicate and ordering cases, existing regression tests, and larger tests if the code is part of a wider system.

### Debugging worksheet

| Field | What to record |
|---|---|
| Expected behavior | Contractually correct result |
| Actual behavior | Observed result or exception |
| Smallest failing input | Minimal reproducible example |
| Relevant state | Variables, pointers, queue, heap, or window |
| Broken invariant | Condition that stopped being true |
| Root cause | Why the incorrect state occurred |
| Fix | Smallest correct change |
| Regression test | Test that fails before the fix and passes after it |

---

## Python debugging tools

### 1. Read tracebacks from the bottom up

A traceback shows the call path that led to an exception. Start with the exception type and message at the bottom, then inspect the relevant frames above it.

~~~python
def average(values):
    return sum(values) / len(values)

def report(values):
    return average(values)

report([])
~~~

This raises **ZeroDivisionError** because the function divides by the length of an empty list. Decide whether empty input is valid and what the function contract requires. Do not automatically suppress exceptions.

The standard-library [traceback module](https://docs.python.org/3/library/traceback.html) can format and inspect exception tracebacks programmatically.

### 2. Use breakpoints with pdb

Python's built-in **breakpoint()** enters the configured debugger. In a normal development session, it commonly enters **pdb**.

~~~python
def find_target(values, target):
    left = 0
    right = len(values) - 1

    while left <= right:
        breakpoint()
        middle = (left + right) // 2

        if values[middle] == target:
            return middle
        if values[middle] < target:
            left = middle + 1
        else:
            right = middle - 1

    return -1
~~~

Useful debugger commands:

| Command | Purpose |
|---|---|
| **p expression** | Evaluate and print an expression |
| **n** | Execute next line without stepping into a called function |
| **s** | Step into a called function |
| **c** | Continue execution |
| **l** | Show nearby source lines |
| **where** | Show the current stack |
| **q** | Quit the debugger |

Start a script under the debugger:

~~~bash
python -m pdb path/to/script.py
~~~

Remove temporary breakpoints before committing code unless an intentional debugger hook is part of the tool.

Reference: [Python pdb documentation](https://docs.python.org/3/library/pdb.html).

### 3. Use logging for useful execution context

Logging is useful when a failure depends on a sequence of events or cannot be reproduced easily in an interactive debugger.

~~~python
import logging

logger = logging.getLogger(__name__)

def process_event(event_id, attempt):
    logger.info(
        "Processing event_id=%s attempt=%s",
        event_id,
        attempt,
    )
~~~

Configure logging at the application entry point:

~~~python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s %(message)s",
)
~~~

Log meaningful context such as a stable request or event identifier, the operation, retry number, state transition, and outcome. Avoid logging secrets, credentials, access tokens, or unnecessary sensitive payloads. Avoid logging every iteration of a large hot loop unless intentional.

Use module-level loggers created with **logging.getLogger(__name__)** to participate in Python's hierarchical logging system.

Reference: [Python logging documentation](https://docs.python.org/3/library/logging.html).

### 4. Use assertions for internal invariants

An assertion documents a condition expected to be true at a particular point.

~~~python
def remove_first(items):
    assert items is not None
    return items.pop(0)
~~~

Assertions are not a substitute for validating untrusted input. Python can omit assertion code when optimization is enabled, so do not use **assert** for checks that must always execute or for required input validation.

Use an explicit exception for a public input contract:

~~~python
def average(values):
    if not values:
        raise ValueError("values must not be empty")
    return sum(values) / len(values)
~~~

Reference: [The Python assert statement](https://docs.python.org/3/reference/simple_stmts.html#the-assert-statement).

---

## Testing a suspected bug

### Test categories

| Category | Example |
|---|---|
| Typical input | Normal list with several values |
| Empty input | Empty list, string, or graph |
| Single element | One item or event |
| Boundary | First index, last index, exact time limit |
| Duplicates | Repeated keys or equal priorities |
| Already ordered | Sorted input |
| Reverse ordered | Descending input |
| Missing target | Target does not occur |
| Invalid input | Wrong type, if contract defines behavior |
| Scale | Input near the stated constraint |

Do not invent invalid-input behavior. Define whether the function rejects such input and how.

### Write regression tests with unittest

The standard-library **unittest** framework supports test cases, assertions, fixtures, and test discovery.

~~~python
import unittest

def first_at_least(values, target):
    left = 0
    right = len(values)
    while left < right:
        middle = (left + right) // 2
        if values[middle] < target:
            left = middle + 1
        else:
            right = middle
    return left

class TestFirstAtLeast(unittest.TestCase):
    def test_empty_input(self):
        self.assertEqual(first_at_least([], 5), 0)

    def test_first_position(self):
        self.assertEqual(first_at_least([2, 4, 6], 1), 0)

    def test_middle_position(self):
        self.assertEqual(first_at_least([2, 4, 6], 5), 2)

    def test_after_last_position(self):
        self.assertEqual(first_at_least([2, 4, 6], 8), 3)

    def test_duplicate_values(self):
        self.assertEqual(first_at_least([1, 3, 3, 3, 7], 3), 1)

if __name__ == "__main__":
    unittest.main()
~~~

Run a test file directly:

~~~bash
python path/to/test_file.py
~~~

Or use discovery from the project root:

~~~bash
python -m unittest
~~~

Use framework assertion methods such as **assertEqual**, **assertTrue**, and **assertRaises**, rather than bare **assert** statements inside tests.

Reference: [Python unittest documentation](https://docs.python.org/3/library/unittest.html).

### Differential testing

When a simple, obviously correct implementation is available, compare it with the optimized implementation over generated inputs.

~~~python
import random

def linear_first_at_least(values, target):
    for index, value in enumerate(values):
        if value >= target:
            return index
    return len(values)

def binary_first_at_least(values, target):
    left = 0
    right = len(values)
    while left < right:
        middle = (left + right) // 2
        if values[middle] < target:
            left = middle + 1
        else:
            right = middle
    return left

rng = random.Random(2026)
for _ in range(1_000):
    values = sorted(rng.randint(-20, 20) for _ in range(rng.randint(0, 30)))
    target = rng.randint(-25, 25)
    expected = linear_first_at_least(values, target)
    actual = binary_first_at_least(values, target)
    assert actual == expected, (values, target, expected, actual)
~~~

This demonstrates deterministic randomized testing. A fixed seed makes the generated sequence reproducible in the same Python random implementation context. Keep discovered failures as explicit regression tests rather than relying only on random generation.

---

## Common algorithmic bugs

### 1. Off-by-one errors

Typical causes include using **<** where **<=** is required, treating an inclusive right boundary as exclusive, returning the wrong pointer after a search, and forgetting that a slice's end index is excluded.

**Prevention:** write down whether each interval is inclusive or half-open. State what the active interval represents before writing binary search.

### 2. Incorrect loop progress

A loop can run forever if an iteration does not move its state toward termination.

**Prevention:** identify the variable that must progress and show why every branch changes it or returns.

### 3. Incorrect visited-node timing in graph traversal

In ordinary BFS, marking a node when it is discovered and enqueued prevents it from being enqueued repeatedly.

~~~python
from collections import deque

def bfs(graph, start):
    visited = {start}
    queue = deque([start])
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    return order
~~~

This example assumes graph keys and neighbor values are hashable. It returns nodes reachable from **start**.

### 4. Stale state in sliding windows

A running count or sum must represent exactly the current window. If an item leaves the window but its contribution is not removed, later results become incorrect.

**Prevention:** define the window invariant and update state where the boundary moves.

### 5. Mutation and aliasing

Assignment does not copy a mutable list:

~~~python
original = [1, 2]
alias = original
alias.append(3)
print(original)  # [1, 2, 3]
~~~

Both names refer to the same list. If a problem requires preserving input, make that part of the contract and choose a suitable copy or non-mutating algorithm.

### 6. Incorrect heap assumptions

Python's **heapq** implements a min-heap: the smallest item is at index zero. Define priority ordering explicitly and test equal priorities.

Reference: [official heapq documentation](https://docs.python.org/3/library/heapq.html).

### 7. Incorrect time-window boundaries

Including timestamps greater than or equal to a cutoff is not the same as including only timestamps strictly greater than it.

**Prevention:** state whether the interval is closed, open, or half-open, and test events exactly at both endpoints.

### 8. Assuming input order

A solution may work on sorted input even though the contract does not promise sorting.

**Prevention:** verify preconditions before using binary search, two pointers, or order-dependent logic. Sort only if allowed and account for its cost and mutation behavior.

### 9. Ignoring recursion limits and input scale

A recursive solution may not be appropriate for a deep input structure.

**Prevention:** estimate maximum recursion depth from input constraints. Consider an iterative traversal when depth can be large. Do not change interpreter recursion limits as a substitute for understanding the algorithm.

### 10. Confusing expected and worst-case complexity

Hash-table operations are commonly expected O(1), not a universal worst-case guarantee. State assumptions behind complexity claims.

---

## A disciplined optimization workflow

Optimization should follow correctness, not replace it.

1. **Establish a baseline.** Record input shape and size, expected output, implementation, relevant tests, and measurement environment.
2. **Analyze complexity.** Look for nested scans, repeated lookups, repeated sorting, unnecessary retained objects, deep recursion, or inefficient top-K selection.
3. **Measure the suspected bottleneck.** Use representative inputs and a repeatable method.
4. **Change one thing at a time.** Examples include indexing repeated lookups, using a deque for queue workloads, using a heap for repeated minimum selection, or avoiding unnecessary intermediate collections.
5. **Re-run correctness tests.** Compare against a simple reference implementation when practical.
6. **Measure again.** Use the same input and environment. Do not claim meaningful improvement when results are small or inconsistent.
7. **Explain the trade-off.** Discuss time, auxiliary space, mutation, ordering, readability, and whether the optimization matters at the expected scale.

| Symptom | Possible cause | Questions to ask |
|---|---|---|
| Runtime grows sharply | Nested scans or repeated work | Can a map, heap, or prefix sum reduce work? |
| Many repeated lookups | Linear search inside a loop | Can data be indexed once? |
| Repeated sorting | Sorting the same data repeatedly | Can sorted state be reused? |
| Memory grows with input | Retaining unnecessary objects | Can data be streamed or discarded? |
| Deep recursion | Input-shaped recursion | Is an iterative approach safer? |
| Slow top-K selection | Sorting all items | Is a heap more suitable when K is small? |

A theoretically better algorithm is not automatically faster for every small input. Complexity analysis is usually the first tool for rejecting approaches that cannot scale.

---

## Measuring runtime and memory

### 1. Use timeit for small code sections

The **timeit** module measures small Python snippets and provides command-line and callable interfaces.

~~~bash
python -m timeit "'-'.join(str(n) for n in range(100))"
~~~

For repeatable comparisons, keep the workload identical:

~~~python
import timeit

def build_with_loop():
    result = []
    for number in range(1_000):
        result.append(number * 2)
    return result

def build_with_comprehension():
    return [number * 2 for number in range(1_000)]

loop_time = timeit.timeit(build_with_loop, number=1_000)
comprehension_time = timeit.timeit(build_with_comprehension, number=1_000)
print("loop:", loop_time)
print("comprehension:", comprehension_time)
~~~

These are measurements from the machine and runtime where the code is executed, not universal performance claims.

Important cautions:
- **timeit** disables garbage collection during timed execution by default. If garbage collection is part of the workload, account for it explicitly.
- Other processes can interfere with timing.
- Repeated measurements can vary.
- Microbenchmarks do not automatically represent full application performance.

Reference: [Python timeit documentation](https://docs.python.org/3/library/timeit.html).

### 2. Use tracemalloc to inspect Python memory allocations

**tracemalloc** can record Python memory allocations, identify allocation locations, and compare snapshots.

~~~python
import tracemalloc

tracemalloc.start()
items = [str(number) for number in range(10_000)]
snapshot = tracemalloc.take_snapshot()

for statistic in snapshot.statistics("lineno")[:5]:
    print(statistic)
~~~

Compare allocations across two points:

~~~python
import tracemalloc

tracemalloc.start()
before = tracemalloc.take_snapshot()

records = []
for number in range(5_000):
    records.append({"id": number, "status": "queued"})

after = tracemalloc.take_snapshot()
for statistic in after.compare_to(before, "lineno")[:5]:
    print(statistic)
~~~

A snapshot comparison reports differences in traced Python allocations. It is not a complete measure of process resident memory or proof of a memory leak. Interpret it in the context of the workload and other process memory. Start tracing early when you need to observe earlier allocations.

Reference: [Python tracemalloc documentation](https://docs.python.org/3/library/tracemalloc.html).

### 3. Profiling is not the same as benchmarking

- **Benchmarking** compares elapsed time for a defined workload.
- **Profiling** helps identify where execution time is spent.
- **Memory tracing** helps inspect traced allocation behavior.
- **Complexity analysis** estimates how work grows with input size.

Use the tool that answers the question you actually have. Do not use a single timing result as proof of complexity.

---

## DevOps and SRE debugging scenarios

These are algorithmic demonstrations, not definitions of production platform behavior.

### Scenario 1 — Retry scheduler fires jobs too early

Investigate whether the scheduler compares current time with next attempt time using the intended boundary, timestamps use consistent units, the heap is ordered by next scheduled time, and failed jobs are rescheduled.

Test just before, exactly at, and just after the scheduled time.

### Scenario 2 — Rolling failure rate is incorrect

Investigate what the window represents, whether the cutoff is inclusive, whether expired events are removed before calculating the rate, and whether the denominator is the event count or a fixed expected count.

Test events at the cutoff and a window with no events.

### Scenario 3 — Dependency traversal misses a service

Investigate graph direction, edge meaning, visited-node timing, and services with no outgoing edges.

Test a chain, branch, cycle, and isolated service.

### Scenario 4 — Alert suppression never expires

Investigate whether the last-alert timestamp is updated only when an alert is emitted, whether the cooldown boundary is correct, whether state is keyed by the right alert identity, and whether obsolete state is cleaned up.

Test immediately before, exactly at, and after the cooldown boundary.

### Scenario 5 — Memory grows during event processing

Investigate which objects remain reachable, whether a collection has a retention limit, whether snapshots are taken at comparable workload points, and whether growth is expected for the input volume.

Use **tracemalloc** snapshots as one diagnostic signal. Do not conclude that every allocation difference is a leak.

Production systems also have concerns such as concurrency, persistence, clock behavior, retries across process restarts, and failure handling.

---

## Hands-on debugging exercises

For each exercise, identify the bug, write a failing test, implement the smallest correct fix, and explain why it works.

### Exercise 1 — Binary search boundary

**Buggy code:**

~~~python
def first_at_least(values, target):
    left = 0
    right = len(values) - 1
    while left < right:
        middle = (left + right) // 2
        if values[middle] < target:
            left = middle + 1
        else:
            right = middle
    return left
~~~

**Contract:** values is sorted ascending. Return the first index whose value is at least target, or len(values) if none exists.

**Fix:**

~~~python
def first_at_least(values, target):
    left = 0
    right = len(values)
    while left < right:
        middle = (left + right) // 2
        if values[middle] < target:
            left = middle + 1
        else:
            right = middle
    return left
~~~

The search interval is half-open: [left, right). The initial right boundary must be len(values). The invariant keeps the first qualifying position within the active interval. Complexity: O(log n) time and O(1) auxiliary space.

Test empty input, target above the final value, target below the first value, and duplicates.

### Exercise 2 — Sliding-window stale state or unnecessary allocation?

**Initial code:**

~~~python
def count_recent(events, cutoff):
    recent = []
    for timestamp in events:
        if timestamp >= cutoff:
            recent.append(timestamp)
    return len(recent)
~~~

**Contract:** events is sorted ascending. Count timestamps greater than or equal to cutoff.

For this contract, the original implementation is correct: it filters every timestamp and counts the retained values. It allocates a second list unnecessarily.

~~~python
def count_recent(events, cutoff):
    return sum(timestamp >= cutoff for timestamp in events)
~~~

This is an optimization, not a correctness fix. Both implementations take O(n) time; the revised version uses O(1) auxiliary space, excluding the input and constant generator state. Debugging should distinguish incorrect behavior from code that is merely less efficient.

### Exercise 3 — Graph traversal duplicate work

**Initial code:**

~~~python
from collections import deque

def reachable_count(graph, start):
    queue = deque([start])
    visited = set()
    while queue:
        node = queue.popleft()
        if node in visited:
            continue
        visited.add(node)
        for neighbor in graph.get(node, []):
            queue.append(neighbor)
    return len(visited)
~~~

This returns the correct count for a finite graph with hashable node identifiers, but a node may be enqueued multiple times before its first dequeue.

**Improved implementation:**

~~~python
from collections import deque

def reachable_count(graph, start):
    visited = {start}
    queue = deque([start])
    while queue:
        node = queue.popleft()
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    return len(visited)
~~~

Mark nodes as discovered when enqueued. Each reachable node is enqueued at most once. With adjacency lists, time is O(V + E), and auxiliary space is O(V).

### Exercise 4 — Mutable alias

**Buggy code:**

~~~python
def append_status(history, status):
    updated = history
    updated.append(status)
    return updated
~~~

**Contract:** return a new list with status appended, leaving history unchanged.

**Fix:**

~~~python
def append_status(history, status):
    updated = history.copy()
    updated.append(status)
    return updated
~~~

Assignment creates another reference to the same list. A shallow copy takes O(n) time and O(n) additional space. If nested mutable objects must also be isolated, define the required copy semantics; a shallow copy is not sufficient.

### Exercise 5 — Cooldown boundary

**Buggy code:**

~~~python
def should_emit(now, last_emitted, cooldown):
    return now - last_emitted > cooldown
~~~

**Contract:** emit when at least cooldown time units have elapsed. Assume consistent units and a non-None last_emitted value.

**Fix:**

~~~python
def should_emit(now, last_emitted, cooldown):
    return now - last_emitted >= cooldown
~~~

“At least” includes equality. Test elapsed time less than, equal to, and greater than the cooldown.

---

## Interview communication framework

When asked to debug or optimize code, narrate the investigation:

1. **Restate the contract:** “The function returns the first qualifying index, including the end position when no value qualifies.”
2. **Reproduce:** “This minimal input demonstrates the failure.”
3. **Inspect:** “The right boundary is initialized as inclusive, but the loop treats it as exclusive.”
4. **Explain the invariant:** “The candidate position remains inside the half-open interval.”
5. **Fix:** “I will align the initial boundary with the interval convention.”
6. **Test:** “I will cover empty input, duplicates, and both sides of the range.”
7. **Analyze:** “The algorithm remains logarithmic in time and constant in auxiliary space.”

For optimization, state the baseline and complexity, measured bottleneck, proposed change, tests showing behavior is preserved, and time/space/readability trade-off.

A strong answer explains why the fix is correct, not just which line changed.

---

## Readiness checklist

- [ ] I can read a traceback and locate the failing operation.
- [ ] I can use **breakpoint()** and inspect program state.
- [ ] I can add useful logging without exposing sensitive data.
- [ ] I know when assertions are appropriate and when explicit validation is required.
- [ ] I can create a minimal reproducible input.
- [ ] I can write a regression test using **unittest**.
- [ ] I can test empty, single-element, duplicate, and boundary cases.
- [ ] I can state and check an algorithm invariant.
- [ ] I can distinguish a correctness bug from an optimization opportunity.
- [ ] I can explain time and auxiliary-space complexity.
- [ ] I can use **timeit** carefully and describe its limitations.
- [ ] I can use **tracemalloc** without confusing traced allocations with total process memory.
- [ ] I can compare an optimized solution with a simple reference implementation.
- [ ] I can explain optimization trade-offs.

---

## References

Official Python documentation consulted for the Python-specific behavior and tools described here:

| Topic | Official reference |
|---|---|
| Interactive debugger | [pdb — The Python Debugger](https://docs.python.org/3/library/pdb.html) |
| Traceback inspection | [traceback — Print or retrieve a stack traceback](https://docs.python.org/3/library/traceback.html) |
| Application logging | [logging — Logging facility for Python](https://docs.python.org/3/library/logging.html) |
| Unit testing | [unittest — Unit testing framework](https://docs.python.org/3/library/unittest.html) |
| Runtime measurement | [timeit — Measure execution time of small code snippets](https://docs.python.org/3/library/timeit.html) |
| Memory allocation tracing | [tracemalloc — Trace memory allocations](https://docs.python.org/3/library/tracemalloc.html) |
| Assertion semantics | [The Python assert statement](https://docs.python.org/3/reference/simple_stmts.html#the-assert-statement) |
| Heap behavior | [heapq — Heap queue algorithm](https://docs.python.org/3/library/heapq.html) |

**Version note:** Python documentation is versioned. Check the documentation matching your interpreter before relying on version-specific debugger, profiler, or command-line features. The pure-Python **profile** module is deprecated in the current Python 3.15 documentation; this guide does not recommend it.

---

## Navigation

- [← Previous: DevOps and SRE Problems](../07%20DevOps%20and%20SRE%20Problems/README.md)
- [↑ Interview Preparation Home](../README.md)
- [Python Coding Interview](../02%20Python%20Coding%20Interview/README.md)
- [Pattern Recognition](../03%20Pattern%20Recognition/README.md)
- [Easy Problem Sets](../04%20Easy%20Problem%20Sets/README.md)
- [Medium Problem Sets](../05%20Medium%20Problem%20Sets/README.md)
- [Advanced Problem Sets](../06%20Advanced%20Problem%20Sets/README.md)
- [DevOps and SRE Problems](../07%20DevOps%20and%20SRE%20Problems/README.md)

**Next:** Timed Coding Assessments will be added in a later step.
