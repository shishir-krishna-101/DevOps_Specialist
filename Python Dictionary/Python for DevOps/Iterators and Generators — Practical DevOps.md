# Iterators and Generators — Practical DevOps

Iterators and generators are useful when DevOps automation needs to process a stream of data without requiring the whole stream to be materialized at once.

> **Source note:** Python behavior in this guide is based on the official Python documentation. The DevOps examples and design recommendations are repository design decisions.

---

## 1. The DevOps mental model

Many automation tasks naturally produce sequences:

```text
log file
  ↓
lines
  ↓
filter
  ↓
transform
  ↓
report / action
```

The important question is often not “How do I build one huge list?” but:

> “How do I consume the next useful item when I need it?”

This is where iteration and generators become valuable.

---

## 2. Iterable vs iterator

An **iterable** is an object from which Python can obtain an iterator.

An **iterator** represents a stream of values and provides `__next__()`. When the stream is exhausted, `__next__()` raises `StopIteration`.

For example:

```python
servers = ["api-1", "api-2", "worker-1"]

iterator = iter(servers)

print(next(iterator))
print(next(iterator))
print(next(iterator))
```

A `for` loop performs this iteration protocol for you, so normally you write:

```python
for server in servers:
    print(server)
```

Official reference:

https://docs.python.org/3/howto/functional.html#iterators

---

## 3. Iterator state

An iterator keeps track of where it currently is in the stream.

```python
items = ["a", "b", "c"]
it = iter(items)

print(next(it))  # a
print(next(it))  # b
print(next(it))  # c
```

After the final value, another `next(it)` raises `StopIteration`.

A useful consequence is that consuming an iterator advances it:

```python
it = iter([1, 2, 3])

print(list(it))
print(list(it))
```

The second conversion sees an already-consumed iterator.

This is different from repeatedly iterating over a reusable container such as a list.

---

## 4. Why this matters in automation

Consider a large command output or log stream.

A list-oriented design might do this:

```python
lines = read_all_lines()
interesting = [line for line in lines if "ERROR" in line]
```

An iterator-oriented design can process values as they arrive:

```python
def errors(lines):
    for line in lines:
        if "ERROR" in line:
            yield line
```

Then:

```python
for line in errors(lines):
    report(line)
```

The generator does not need to construct a separate list of every matching line.

This is a repository design example, not a claim that generators are always faster or use less memory in every workload.

---

## 5. Generator functions

A function containing `yield` is a generator function.

```python
def environments(values):
    for value in values:
        yield value.strip().lower()
```

Calling it returns a generator object rather than immediately producing every result:

```python
items = environments([" DEV ", "PROD "])

print(next(items))
print(next(items))
```

The generator resumes where it previously yielded and preserves its local state.

Official references:

https://docs.python.org/3/howto/functional.html#generators

https://docs.python.org/3/reference/expressions.html#yield-expressions

---

## 6. Generator expressions

A generator expression provides a compact way to create a generator:

```python
servers = ["api-1", "worker-1", "api-2"]

api_servers = (server for server in servers if server.startswith("api-"))

for server in api_servers:
    print(server)
```

Compare this with a list comprehension:

```python
api_servers = [
    server for server in servers
    if server.startswith("api-")
]
```

The list comprehension creates a list immediately. The generator expression produces values during iteration.

Official reference:

https://docs.python.org/3/reference/expressions.html#generator-expressions

---

## 7. Reading files as streams

File objects are iterable.

This makes line-oriented processing natural:

```python
from pathlib import Path

with Path("application.log").open(encoding="utf-8") as log_file:
    for line in log_file:
        if "ERROR" in line:
            print(line.rstrip())
```

This pattern is useful when processing logs, manifests, reports, or other line-oriented data.

Do not assume that every file format can be safely processed line-by-line. The parsing strategy must match the format.

---

## 8. Generator pipelines

Generators can be composed into stages:

```python
def non_empty(lines):
    for line in lines:
        line = line.strip()
        if line:
            yield line


def errors(lines):
    for line in lines:
        if "ERROR" in line:
            yield line


with open("application.log", encoding="utf-8") as log_file:
    pipeline = errors(non_empty(log_file))

    for line in pipeline:
        print(line)
```

Conceptually:

```text
file
 ↓
non_empty()
 ↓
errors()
 ↓
consumer
```

Each stage can remain focused on one transformation.

---

## 9. `yield from`

`yield from` delegates iteration to another iterable or generator.

```python
def all_servers(groups):
    for group in groups:
        yield from group
```

Example:

```python
groups = [
    ["api-1", "api-2"],
    ["worker-1", "worker-2"],
]

for server in all_servers(groups):
    print(server)
```

This can make generator composition easier to read.

Official reference:

https://docs.python.org/3/reference/expressions.html#yield-expressions

---

## 10. Generators are single-pass streams

A generator is normally consumed as it is iterated.

```python
def values():
    yield 1
    yield 2
    yield 3

items = values()

print(list(items))
print(list(items))
```

The second operation does not recreate the generator's values.

If a workflow needs repeated independent passes, a reusable container or a function that creates a new iterator may be more appropriate.

---

## 11. `itertools`

The standard-library `itertools` module provides iterator building blocks.

Official documentation:

https://docs.python.org/3/library/itertools.html

Useful examples include:

```python
from itertools import chain, islice

sources = [
    ["api-1", "api-2"],
    ["worker-1"],
]

for server in chain.from_iterable(sources):
    print(server)
```

And limiting a stream:

```python
from itertools import islice

first_five = islice(server_stream, 5)

for server in first_five:
    print(server)
```

`itertools` also provides tools for grouping, filtering, combining, batching, and other iterator operations.

---

## 12. Batching data

When an API, database, or other boundary requires bounded groups of work, batching can be useful.

On Python versions where `itertools.batched()` is available:

```python
from itertools import batched

servers = ["api-1", "api-2", "api-3", "api-4", "api-5"]

for batch in batched(servers, 2):
    process_batch(batch)
```

The official documentation describes `batched()` as consuming the input lazily and yielding batches as they are formed.

**Version note:** `itertools.batched()` was added in Python 3.12. If the repository targets an older Python version, use an implementation appropriate to that supported version rather than assuming this function exists.

---

## 13. Infinite iterators

Some iterator tools can produce unbounded streams.

For example:

```python
from itertools import count, islice

for value in islice(count(1), 5):
    print(value)
```

The important lesson is that an unbounded iterator needs a bounded consumer when the program is expected to terminate.

The official `itertools` documentation identifies several functions that can produce infinite streams.

---

## 14. Processing command output

A practical subprocess pattern is to treat output as a stream when the API being used provides an iterable stream of lines.

For example, with `subprocess.Popen`:

```python
import subprocess

process = subprocess.Popen(
    ["your-command"],
    stdout=subprocess.PIPE,
    text=True,
)

assert process.stdout is not None

for line in process.stdout:
    line = line.rstrip("\n")
    process_line(line)

return_code = process.wait()
```

The actual command must come from the tool's authoritative documentation or the system being automated. `your-command` is intentionally a placeholder.

For simpler subprocess use cases, prefer `subprocess.run()` as covered in the earlier `subprocess` topic.

Official reference:

https://docs.python.org/3/library/subprocess.html

---

## 15. Lazy does not mean free

Generators can avoid materializing a complete sequence, but they do not eliminate the work required to produce each value.

For example:

```python
def transformed(values):
    for value in values:
        yield expensive_transform(value)
```

The transformation still occurs when the generator is consumed.

Also remember that downstream operations can materialize the stream:

```python
items = list(transformed(values))
```

This is sometimes correct, but it removes the streaming property at that point.

---

## 16. Exceptions in generators

Exceptions raised while a generator is being consumed occur during iteration.

```python
def parse_values(values):
    for value in values:
        yield int(value)
```

The conversion error is encountered when the generator advances to the problematic value.

Therefore, error handling should be placed at the boundary where the application can make an appropriate decision:

```python
try:
    for value in parse_values(raw_values):
        process(value)
except ValueError as exc:
    report_failure(exc)
```

Do not hide parsing failures merely to keep a pipeline moving unless that behavior is an explicit requirement.

---

## 17. Backpressure and bounded work

Python's iterator protocol does not by itself provide a complete queueing or backpressure system.

For automation pipelines, however, lazy consumption can help keep each stage from eagerly constructing the entire downstream dataset.

If the workflow needs explicit producer/consumer coordination, bounded queues, cancellation, or concurrency, use the appropriate concurrency primitives rather than assuming a generator solves those requirements.

This distinction matters in production automation.

---

## 18. When to use a list instead

Use a list or another materialized collection when you genuinely need collection semantics such as:

- repeated iteration;
- indexing;
- slicing;
- random access;
- knowing the complete set before continuing;
- passing the complete collection to an API that requires it.

Use an iterator or generator when sequential consumption is sufficient and lazy production makes the design clearer.

Do not choose generators solely because they sound more advanced.

---

## 19. Practical DevOps pattern: filter → transform → act

A useful automation shape is:

```python
def healthy_servers(records):
    for record in records:
        if record["status"] == "healthy":
            yield record["name"]


for server in healthy_servers(server_records):
    notify(server)
```

The generator handles selection and transformation. The consumer decides what action to take.

This separation makes it easier to test the transformation independently from the side effect.

---

## 20. Practical DevOps pattern: streaming validation

Suppose a configuration export contains many records.

Instead of first constructing every validation result:

```python
def invalid_records(records):
    for record in records:
        if not is_valid(record):
            yield record
```

The caller can stop after finding the first invalid record:

```python
first_invalid = next(invalid_records(records), None)
```

Or report every invalid record:

```python
for record in invalid_records(records):
    report(record)
```

The consumer determines how much of the stream is needed.

---

## 21. Practical DevOps pattern: reusable pipeline stages

Keep stages small:

```python
def normalize(lines):
    for line in lines:
        yield line.strip().lower()


def only_errors(lines):
    for line in lines:
        if "error" in line:
            yield line


def unique(lines):
    seen = set()
    for line in lines:
        if line not in seen:
            seen.add(line)
            yield line
```

Then compose them:

```python
pipeline = unique(only_errors(normalize(lines)))

for line in pipeline:
    print(line)
```

The `unique()` stage intentionally keeps state. That means it may grow with the number of distinct values, so “generator” does not automatically mean “constant memory.”

---

## 22. Common mistakes

### Mistake 1: treating an iterator like a reusable list

```python
items = iter([1, 2, 3])

first = list(items)
second = list(items)
```

The second result is empty because the iterator was consumed.

### Mistake 2: accidentally materializing everything

```python
results = list(huge_generator())
```

This may defeat the reason for using a generator.

### Mistake 3: using a generator when repeated access is required

If callers need indexing or repeated traversal, a list may be the clearer choice.

### Mistake 4: hiding exceptions

Do not silently discard malformed records simply because the data is being streamed.

### Mistake 5: assuming generators provide concurrency

Generators control iteration. They do not automatically create threads, processes, or asynchronous execution.

### Mistake 6: creating unbounded pipelines without a stopping condition

Be especially careful with infinite iterators.

---

## 23. Exercise: log filtering pipeline

Build a generator pipeline that:

1. accepts an iterable of log lines;
2. strips surrounding whitespace;
3. ignores empty lines;
4. selects lines containing `ERROR`;
5. yields the matching lines one at a time.

Then consume the generator with a `for` loop.

### Extension
Add a stage that extracts a simple service name from each matching line.

Keep parsing rules explicit rather than assuming a log format that was not specified.

---

## 24. Exercise: bounded batch processing

Create a stream of deployment targets and process them in batches.

Requirements:

- the source should be iterable;
- processing should happen batch-by-batch;
- the complete target list should not be required by the consumer;
- the batch size should be configurable.

If using `itertools.batched()`, document the minimum Python version required by the repository.

---

## 25. Exercise: first matching resource

Write a generator that yields resources satisfying a condition.

Then use `next()` with a default value to obtain the first match without materializing all matches.

Consider:

```text
resource stream
      ↓
filter
      ↓
first match
```

Test both the matching and no-match cases.

---

## 26. Knowledge checks

Before moving on, you should be able to answer:

1. What is the difference between an iterable and an iterator?
2. What does `next()` do when an iterator is exhausted?
3. Why does a generator preserve local state between `yield` operations?
4. When is a generator expression preferable to a list comprehension?
5. Why can a generator still consume significant memory?
6. What does `yield from` do?
7. Why are some `itertools` functions unsafe to consume without a limit?
8. When should a DevOps script prefer a list over a generator?
9. Why does a generator not automatically provide concurrency?
10. Where should an automation pipeline handle an exception raised during iteration?

---

## 27. Completion criteria

You have completed this topic when you can:

- explain iterable vs iterator;
- use `iter()` and `next()` correctly;
- write generator functions with `yield`;
- use generator expressions;
- compose generator-based processing stages;
- explain why iterators are commonly single-pass;
- use appropriate `itertools` building blocks;
- recognize infinite-iterator risks;
- choose between streaming and materialization deliberately;
- separate transformation stages from external side effects.

---

## 28. Verified facts vs repository design

### Verified facts

- Python iterators implement the iterator protocol and raise `StopIteration` when exhausted.
- Generator functions contain `yield` and return generator objects when called.
- Generator execution can suspend at `yield` and resume later.
- Generator expressions produce generator objects.
- `itertools` provides iterator-building functions, including tools that may produce infinite streams.
- `itertools.batched()` was added in Python 3.12.

### Repository design decisions

- DevOps examples emphasize logs, command output, configuration records, and automation pipelines.
- The guide recommends separating transformation stages from side effects.
- Exercises are intentionally designed around automation workflows.

### Demonstrations

Examples using names such as `server_stream`, `process_batch`, `notify`, and `application.log` are teaching examples. They are not claims about any particular infrastructure platform or command-line tool.

---

## References

1. Python Functional Programming HOWTO — Iterators and generators  
   https://docs.python.org/3/howto/functional.html

2. Python `itertools` documentation  
   https://docs.python.org/3/library/itertools.html

3. Python expression reference — `yield` expressions and generator expressions  
   https://docs.python.org/3/reference/expressions.html#yield-expressions

4. Python expression reference — generator expressions  
   https://docs.python.org/3/reference/expressions.html#generator-expressions

5. Python `subprocess` documentation  
   https://docs.python.org/3/library/subprocess.html
