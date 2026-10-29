# Concurrency and Asyncio

Concurrency is about structuring work so that multiple operations can make progress during the same period. Python provides several concurrency models, and choosing between them depends on the workload and the execution model you need.

For DevOps automation, the most useful question is usually not **“How do I make this concurrent?”** but **“What kind of work am I coordinating?”**

> **Verified fact:** Python's standard library documents `threading`, `multiprocessing`, and `concurrent.futures` under concurrent execution. `asyncio` provides asynchronous programming with `async`/`await` and is commonly suited to I/O-bound, high-level network code.

---

## 1. Concurrency vs Parallelism

These terms are related but not identical.

- **Concurrency**: multiple tasks are in progress during overlapping periods.
- **Parallelism**: multiple tasks execute at the same time on different execution resources.

A useful mental model:

```text
Concurrency
    ├── Threads
    ├── asyncio tasks
    └── Processes

Parallelism
    └── Multiple execution resources working at the same time
```

The Python documentation describes concurrent execution tools in terms of workload characteristics such as CPU-bound versus I/O-bound work and execution styles such as event-driven cooperative multitasking versus preemptive multitasking.

**Repository design decision:** This guide focuses on practical selection rather than treating one concurrency model as universally better.

---

## 2. Start With the Workload

Before choosing an API, classify the work.

### I/O-bound work

The program spends substantial time waiting for external operations such as:

- network responses
- file operations
- subprocesses
- external services

Examples in DevOps:

- querying several independent HTTP endpoints
- checking the state of multiple services
- waiting for several independent commands
- collecting data from multiple systems

Threads or asynchronous I/O can be useful for these workloads, depending on the APIs involved.

### CPU-bound work

The program spends substantial time doing computation.

Examples:

- CPU-heavy data transformation
- expensive calculations
- processing large amounts of data in Python code

For CPython, the official `threading` documentation explains that the Global Interpreter Lock limits execution of Python bytecode by multiple threads at the same time. The same documentation points to `multiprocessing` or `ProcessPoolExecutor` when the goal is to make better use of multiple processors for CPU-heavy work.

> **Verified fact:** The GIL discussion above is specific to CPython's documented behavior; do not generalize it to every Python implementation.

---

## 3. The Main Standard-Library Choices

| Tool | Mental model | Typical fit |
|---|---|---|
| `threading` | Multiple threads in one process | I/O-bound work and shared-memory coordination |
| `multiprocessing` | Separate processes | CPU-oriented parallel work and process isolation |
| `concurrent.futures` | High-level executor interface | Submit work and collect results with threads or processes |
| `asyncio` | Cooperative asynchronous tasks | I/O-bound asynchronous/network programs |

The table is a learning aid, not a hard rule. The actual choice depends on the workload, libraries being called, failure behavior, and operational constraints.

---

## 4. Threads With `threading`

The `threading` module provides higher-level interfaces for running multiple threads within a process.

A minimal example:

```python
import threading


def check_service(name):
    print(f"checking {name}")


thread = threading.Thread(target=check_service, args=("api",))
thread.start()
thread.join()
```

### Important concepts

- `Thread` represents a thread of execution.
- `start()` begins execution of the thread.
- `join()` waits for the thread to finish.
- Threads in one process share memory.

### Shared state

Shared memory can be useful, but it also creates synchronization problems.

For example, several threads updating the same mutable object can require synchronization. Python's `threading` module provides synchronization primitives such as locks, reentrant locks, conditions, semaphores, events, and barriers.

**Operational lesson:** Shared mutable state increases the amount of reasoning required. Prefer independent work and explicit communication when practical.

---

## 5. Thread Pools With `ThreadPoolExecutor`

For many automation tasks, `concurrent.futures.ThreadPoolExecutor` provides a simpler abstraction than manually creating and joining threads.

```python
from concurrent.futures import ThreadPoolExecutor


def check_service(name):
    return f"checked {name}"


services = ["api", "worker", "scheduler"]

with ThreadPoolExecutor(max_workers=3) as executor:
    results = list(executor.map(check_service, services))

print(results)
```

`concurrent.futures` provides a high-level interface for asynchronously executing callables. Its executor abstraction includes thread and process implementations.

### Why this is useful in DevOps

A pool can make independent work easier to express:

```text
input items
    ↓
executor
    ├── task A
    ├── task B
    ├── task C
    └── task D
    ↓
results
```

**Common mistake:** Do not assume that increasing the worker count always improves performance. External systems can have rate limits, connection limits, capacity constraints, or their own concurrency controls.

---

## 6. Processes With `multiprocessing`

`multiprocessing` provides process-based concurrency with an API similar in several ways to `threading`.

A basic example:

```python
from multiprocessing import Process


def worker(name):
    print(f"working on {name}")


if __name__ == "__main__":
    process = Process(target=worker, args=("job-1",))
    process.start()
    process.join()
```

The `if __name__ == "__main__":` guard is important in multiprocessing examples because child-process startup behavior differs across platforms.

### Why processes matter

Separate processes do not share the same memory space in the way threads in one process do. The official documentation describes `multiprocessing` as a way to effectively sidestep the GIL by using subprocesses and to leverage multiple processors.

This makes processes a useful option for CPU-heavy Python work.

### Process pools

`multiprocessing.Pool` provides a convenient way to distribute work across processes. `concurrent.futures.ProcessPoolExecutor` provides a higher-level executor interface for process-based work.

---

## 7. `concurrent.futures`: A Useful Abstraction

`concurrent.futures` lets you write code around an executor abstraction instead of coupling the application logic directly to threads or processes.

```python
from concurrent.futures import ThreadPoolExecutor


def transform(value):
    return value * 2


with ThreadPoolExecutor() as executor:
    futures = [executor.submit(transform, value) for value in range(4)]

    for future in futures:
        print(future.result())
```

A `Future` represents work that has been submitted and whose result can be obtained later.

The same high-level style can be used with `ProcessPoolExecutor` when process-based execution is appropriate.

> **Verified fact:** `concurrent.futures.Future` is distinct from `asyncio.Future`; the Python documentation explicitly warns not to confuse the two.

---

## 8. Asyncio Mental Model

`asyncio` provides asynchronous programming using the `async`/`await` syntax.

A minimal example:

```python
import asyncio


async def check_service():
    await asyncio.sleep(1)
    return "ok"


async def main():
    result = await check_service()
    print(result)


asyncio.run(main())
```

The important mental model is cooperative execution:

```text
async task A ──┐
               ├── event loop ──> task progress
async task B ──┤
               └── task progress
```

A coroutine can suspend at an `await`, allowing other scheduled asynchronous work to make progress.

> **Verified fact:** The Python documentation describes `asyncio` as a library for writing concurrent code with `async`/`await` and says it is often a good fit for I/O-bound, high-level network code.

---

## 9. Coroutines

An `async def` function defines a coroutine function.

```python
async def fetch_status():
    ...
```

Calling it produces a coroutine object; it does not execute the coroutine body in the same way as an ordinary function call.

The coroutine must be awaited or otherwise scheduled as part of an asynchronous program.

```python
async def main():
    status = await fetch_status()
    print(status)
```

### Key distinction

```text
normal function
    call → execute

coroutine function
    call → coroutine object
    await/schedule → execute as part of async program
```

---

## 10. Tasks

An asyncio task wraps a coroutine and schedules it for execution by the event loop.

```python
import asyncio


async def check(name):
    await asyncio.sleep(1)
    return name


async def main():
    task_a = asyncio.create_task(check("api"))
    task_b = asyncio.create_task(check("worker"))

    result_a = await task_a
    result_b = await task_b

    print(result_a, result_b)


asyncio.run(main())
```

The important difference is between awaiting operations sequentially and scheduling independent work so that it can make progress concurrently.

**Common mistake:** Creating many tasks without any concurrency limit can overwhelm the local process or the external systems being called.

---

## 11. Waiting for Multiple Async Operations

When several asynchronous operations are independent, `asyncio.gather()` can be useful.

```python
import asyncio


async def check(name):
    await asyncio.sleep(1)
    return f"{name}: ok"


async def main():
    results = await asyncio.gather(
        check("api"),
        check("worker"),
        check("scheduler"),
    )
    print(results)


asyncio.run(main())
```

This is a useful pattern for fan-out/fan-in workflows:

```text
             ┌── check api ─────┐
request ─────┼── check worker ───┼── results
             └── check scheduler┘
```

**Repository design decision:** Examples use small, deterministic operations so the concurrency structure is visible without depending on external services.

---

## 12. Asyncio and Blocking Code

One of the most important asyncio rules is:

> Do not accidentally perform long-running blocking work directly in the event-loop thread.

For example, inserting blocking work into an async function can prevent other tasks from making progress.

Conceptually:

```text
GOOD
async task → await non-blocking operation → other tasks can progress

RISKY
async task → blocking operation → event-loop progress is delayed
```

If a library exposes only blocking APIs, consider whether threads, processes, or an appropriate async integration is a better fit instead of simply wrapping the blocking call inside `async def`.

---

## 13. Cancellation and Time Limits

Long-running automation should account for operations that take too long or need to stop.

Asyncio provides APIs for task cancellation and timeouts. These mechanisms should be designed together with cleanup and failure handling.

A cancellation-aware design should answer:

1. What work can be cancelled?
2. What resources must be cleaned up?
3. What state may have already changed?
4. Should partial results be retained?
5. What should the caller report?

**Operational lesson:** A timeout is not automatically the same thing as successfully stopping the underlying external operation. Always verify the behavior of the API or command being controlled.

---

## 14. Async Context Managers

Async code can use asynchronous context managers with `async with`.

```python
async with resource() as connection:
    await connection.run()
```

The same resource-lifecycle principle applies as with normal context managers: acquire the resource, perform work, and ensure cleanup.

This becomes important when working with asynchronous network clients, streams, and other async resources.

---

## 15. Choosing Between Threads, Processes, and Asyncio

Use this as a starting decision tree:

```text
What kind of work is it?
        │
        ├── Mostly I/O-bound
        │       │
        │       ├── Existing API is blocking
        │       │       → threads may be practical
        │       │
        │       └── API is async-native
        │               → asyncio may be practical
        │
        └── Mostly CPU-bound Python work
                │
                └── consider processes / ProcessPoolExecutor
```

Then ask:

- Is the library blocking or async-native?
- Does the work need shared memory?
- Can the work be safely isolated?
- What are the external service limits?
- How should failures be propagated?
- How will cancellation work?
- How much concurrency is safe?

There is no universal concurrency API for every DevOps task.

---

## 16. DevOps Patterns

### Parallel service checks

```text
service list
    ↓
bounded workers
    ↓
health checks
    ↓
aggregate status
```

Useful when checks are independent and the target systems can handle the concurrency.

### Parallel inventory collection

Collecting independent metadata from multiple systems can benefit from concurrent execution when the APIs are I/O-bound.

### Controlled polling

Polling several resources concurrently can reduce unnecessary serial waiting, but the polling frequency and concurrency should respect service limits.

### CPU-heavy local processing

If a pipeline retrieves data and then performs CPU-heavy Python processing, separate the I/O stage from the CPU stage and choose concurrency models for each stage deliberately.

---

## 17. Concurrency Safety Checklist

Before shipping concurrent automation, ask:

- [ ] Is the workload I/O-bound or CPU-bound?
- [ ] Is the selected API compatible with the libraries being called?
- [ ] Is concurrency bounded?
- [ ] Are shared mutable objects protected or avoided?
- [ ] Are exceptions observed and propagated?
- [ ] Are timeouts defined where appropriate?
- [ ] Is cancellation handled safely?
- [ ] Can partial work leave external systems in an inconsistent state?
- [ ] Are external rate limits respected?
- [ ] Are logs understandable when operations interleave?
- [ ] Can the behavior be tested deterministically?

---

## 18. Common Mistakes

### Mistake 1: Using concurrency without measuring the workload

Concurrency adds complexity. First identify the bottleneck.

### Mistake 2: Treating threads as a universal CPU-performance solution

For CPython, the documented GIL behavior limits Python-bytecode execution by multiple threads. Consider process-based execution for suitable CPU-bound work.

### Mistake 3: Blocking the asyncio event loop

An `async def` function does not automatically make every operation inside it asynchronous.

### Mistake 4: Creating unbounded work

Submitting thousands of operations at once can move the bottleneck from your program to the target service, network, or operating system.

### Mistake 5: Ignoring exception behavior

Concurrent failures can be less obvious than sequential failures. Make result and exception handling explicit.

### Mistake 6: Sharing too much mutable state

Shared state creates synchronization and correctness problems. Prefer isolated work and explicit data flow when possible.

### Mistake 7: Assuming a timeout undoes external side effects

Stopping your local wait does not necessarily reverse a command or API operation already sent to another system.

---

## 19. Practical Exercises

### Exercise 1 — Thread pool

Build a script that checks several local or simulated services concurrently with `ThreadPoolExecutor`.

Requirements:

- use a bounded worker pool
- return one result per service
- handle one failed check without hiding the failure
- produce a final summary

### Exercise 2 — Process pool

Create a CPU-heavy pure-Python function and compare sequential execution with `ProcessPoolExecutor`.

Focus on correctness first; do not claim a speedup unless you actually measure it.

### Exercise 3 — Async fan-out

Create several coroutines that simulate independent network waits using `asyncio.sleep()` and run them concurrently.

Observe the difference between sequential `await` calls and scheduled concurrent tasks.

### Exercise 4 — Cancellation

Create a long-running coroutine and practice cancelling it. Document which cleanup actions still need to occur.

### Exercise 5 — Choose the model

For each task, explain your choice:

1. Call ten independent blocking APIs.
2. Process CPU-heavy data transformations.
3. Coordinate many async-native network operations.
4. Run a small number of independent shell commands.

The goal is not to memorize one answer; explain the workload and constraints.

---

## 20. Knowledge Check

1. What is the difference between concurrency and parallelism?
2. When is threading a practical choice?
3. Why can processes help with CPU-bound Python work in CPython?
4. What does `ThreadPoolExecutor` abstract?
5. What is a `Future`?
6. What does `async def` define?
7. What does `await` allow an async task to do?
8. What is an asyncio task?
9. Why can blocking code be dangerous inside an event-loop thread?
10. Why should concurrent work usually be bounded?
11. When might `ProcessPoolExecutor` be preferable to `ThreadPoolExecutor`?
12. Why does a local timeout not necessarily undo an external side effect?

---

## 21. Completion Checklist

You are ready to move on when you can:

- [ ] Explain concurrency vs parallelism.
- [ ] Classify workloads as primarily I/O-bound or CPU-bound.
- [ ] Create and join a thread.
- [ ] Use `ThreadPoolExecutor` for independent work.
- [ ] Explain the purpose of `multiprocessing`.
- [ ] Use `ProcessPoolExecutor` for suitable CPU-heavy tasks.
- [ ] Explain `Future` at a high level.
- [ ] Write a basic `asyncio` program.
- [ ] Explain coroutines, `await`, and tasks.
- [ ] Run independent coroutines concurrently.
- [ ] Recognize blocking code in async programs.
- [ ] Design around timeouts, cancellation, failures, and bounded concurrency.
- [ ] Choose a concurrency model based on workload and operational constraints.

---

## Verified Facts vs Repository Design

### Verified facts

The technical descriptions in this guide are based on the official Python documentation for concurrent execution, threading, multiprocessing, `concurrent.futures`, and `asyncio`.

### Repository design decisions

- Examples are intentionally small and deterministic.
- DevOps examples use simulated work rather than requiring access to real infrastructure.
- The decision tree is a teaching aid, not a Python specification.
- Performance claims are intentionally avoided unless measured in a specific experiment.

### Demonstration / experiment

Any exercise that compares sequential and concurrent execution should be treated as an experiment. Results depend on the workload, hardware, Python implementation, operating system, and external systems involved.

---

## Authoritative References

- Python — Concurrent Execution: https://docs.python.org/3/library/concurrency.html
- Python — `threading`: https://docs.python.org/3/library/threading.html
- Python — `multiprocessing`: https://docs.python.org/3/library/multiprocessing.html
- Python — `concurrent.futures`: https://docs.python.org/3/library/concurrent.futures.html
- Python — `asyncio`: https://docs.python.org/3/library/asyncio.html
- Python — `asyncio` Tasks and Coroutines: https://docs.python.org/3/library/asyncio-task.html
- Python — `asyncio` Runners: https://docs.python.org/3/library/asyncio-runner.html

For version-specific behavior, consult the Python documentation for the version you are using.
