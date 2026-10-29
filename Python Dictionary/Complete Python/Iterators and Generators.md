# Iterators and Generators

Python's iterator and generator model is one of the most useful abstractions for automation code. It lets a program consume values one at a time instead of requiring every value to be materialized into a collection first.

This topic builds on **Scopes and Python Data Model** and prepares you for later topics such as advanced collections, functional tools, concurrency, and asynchronous programming.

> **Source note:** The technical behavior described here is based primarily on the official Python documentation. Examples marked as demonstrations are repository teaching examples, not claims about an external system.

## 1. The core mental model

An **iterable** is an object from which Python can obtain an iterator.

An **iterator** is an object that produces values one at a time through the iterator protocol.

A **generator** is a convenient kind of iterator produced by a generator function or generator expression.

A useful mental model is:

```text
Iterable
   │
   │ iter(...)
   ▼
Iterator
   │
   │ next(...)
   ▼
next value
   │
   ├── next value
   ├── next value
   └── StopIteration
```

The important distinction is:

```text
iterable → can provide an iterator
iterator → produces the next value
```

Not every iterable is itself an iterator.

## 2. Iterable versus iterator

Lists are iterable:

```python
servers = ["api-1", "api-2", "worker-1"]
```

You can obtain an iterator from the list:

```python
iterator = iter(servers)
```

The iterator produces values with `next()`:

```python
print(next(iterator))  # api-1
print(next(iterator))  # api-2
print(next(iterator))  # worker-1
```

After the iterator is exhausted, another call to `next()` raises `StopIteration`.

```python
try:
    print(next(iterator))
except StopIteration:
    print("No more servers")
```

The list and the iterator are different objects with different roles.

## 3. The iterator protocol

The iterator protocol is based on two methods:

```python
__iter__()
__next__()
```

An iterator's `__iter__()` method returns the iterator itself.

Its `__next__()` method returns the next item. When there are no more items, it raises `StopIteration`.

This is what allows iterators to work with `for` loops and other iteration constructs.

Conceptually:

```python
iterator = iter(some_iterable)

while True:
    try:
        item = next(iterator)
    except StopIteration:
        break

    # process item
```

A normal `for` loop performs this protocol handling for you.

## 4. What a `for` loop is doing conceptually

When you write:

```python
for server in servers:
    print(server)
```

you should understand it as an iteration process rather than as a special operation that only works with lists.

Conceptually, Python obtains an iterator and repeatedly requests its next value until iteration ends.

This explains why many different objects can be used in a `for` loop:

```python
for server in ["api-1", "api-2"]:
    ...

for line in open("deployment.log"):
    ...

for key in {"region": "ap-south-1"}:
    ...
```

Each object provides iteration behavior.

> **Teaching model:** The code above is a conceptual explanation of the iteration protocol. It is not intended to reproduce every implementation detail of the `for` statement.

## 5. `iter()` and `next()`

The built-in `iter()` obtains an iterator from an iterable.

```python
regions = ["ap-south-1", "us-east-1", "eu-west-1"]
iterator = iter(regions)
```

The built-in `next()` asks an iterator for its next value:

```python
print(next(iterator))
print(next(iterator))
```

The iterator maintains its position between calls.

```text
regions
   │
   ▼
iterator
   │
   ├── next() → ap-south-1
   ├── next() → us-east-1
   └── next() → eu-west-1
```

Once exhausted, the iterator raises `StopIteration`.

## 6. `StopIteration` is the normal end of iteration

`StopIteration` is part of the iterator protocol. It signals that the iterator has no more values.

You normally do not need to catch it yourself when using a `for` loop:

```python
for region in regions:
    print(region)
```

Python handles the end-of-iteration condition for you.

When manually driving an iterator with `next()`, you may need to account for it:

```python
iterator = iter(regions)

while True:
    try:
        region = next(iterator)
    except StopIteration:
        break

    print(region)
```

An important iterator rule is that after an iterator's `__next__()` raises `StopIteration`, subsequent calls must continue to raise `StopIteration`.

## 7. Iterators are stateful

An iterator normally represents a current position in a sequence or stream of values.

For example:

```python
commands = ["build", "test", "deploy"]
iterator = iter(commands)

first = next(iterator)
second = next(iterator)
```

The iterator has advanced past the values it returned.

This means that iterators are generally **consumable**.

```python
iterator = iter([1, 2, 3])

print(list(iterator))  # [1, 2, 3]
print(list(iterator))  # []
```

The first `list()` consumes the iterator. The second receives no remaining values.

This is different from asking a reusable iterable such as a list for a fresh iterator.

```python
values = [1, 2, 3]

print(list(values))
print(list(values))
```

Both operations can iterate over the list because the list can provide a new iterator.

## 8. Building a custom iterable

A class can provide an `__iter__()` method that returns an iterator.

For example:

```python
class DeploymentTargets:
    def __init__(self, targets):
        self.targets = targets

    def __iter__(self):
        return iter(self.targets)


targets = DeploymentTargets(["api", "worker", "scheduler"])

for target in targets:
    print(target)
```

Here, `DeploymentTargets` is an iterable because its `__iter__()` method provides an iterator.

This design can be useful when a domain object should expose iteration without exposing all of its internal implementation details.

## 9. A custom iterator

An object can also implement the iterator protocol directly.

```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration

        value = self.current
        self.current -= 1
        return value


for value in Countdown(3):
    print(value)
```

The important pieces are:

- `__iter__()` returns `self` because the object is itself the iterator;
- `__next__()` returns the next value;
- `StopIteration` marks the end.

Custom iterator classes are useful when iteration requires explicit state or protocol behavior that is clearer as a class.

However, many custom iteration problems are simpler to express with a generator.

## 10. Generator functions

A function containing a `yield` expression is a **generator function**.

```python
def deployment_steps():
    yield "validate"
    yield "build"
    yield "deploy"
```

Calling the generator function does not immediately execute the body to completion. It returns a generator iterator.

```python
steps = deployment_steps()
```

Values are produced as the generator is iterated:

```python
print(next(steps))  # validate
print(next(steps))  # build
print(next(steps))  # deploy
```

After the generator reaches the end of the function, iteration is complete.

## 11. `yield` suspends generator execution

Consider:

```python
def numbers():
    print("before first yield")
    yield 1
    print("before second yield")
    yield 2
    print("finished")
```

Creating the generator does not run the function body to completion:

```python
values = numbers()
```

Execution starts when the generator is advanced:

```python
print(next(values))
```

The generator executes until the first `yield`, returns that value, and suspends its execution state.

A later `next()` resumes execution from the suspended point.

The official documentation describes the suspended state as retaining relevant local state and execution information so that the generator can continue where it left off.

## 12. Generator functions preserve local state

This is one reason generators are useful for stateful streams.

```python
def deployment_numbers():
    number = 1

    while number <= 3:
        yield number
        number += 1
```

The local variable `number` remains available when execution resumes after each `yield`.

```python
for number in deployment_numbers():
    print(number)
```

Output:

```text
1
2
3
```

The important idea is not that a generator stores a complete list of results. Instead, its execution can pause and resume while retaining the generator's state.

## 13. Generators versus building a list

Suppose you want to process a sequence of generated values.

A list-producing function might be written as:

```python
def deployment_ids():
    return [f"deployment-{number}" for number in range(1, 4)]
```

A generator can produce the values incrementally:

```python
def deployment_ids():
    for number in range(1, 4):
        yield f"deployment-{number}"
```

The generator is useful when the consumer can process one value at a time.

This can reduce the need to materialize an entire result collection at once.

> **Important:** Do not turn this into a blanket claim that generators are always faster or always use less memory. The benefit depends on the workload, the consumer, and whether all values ultimately need to be materialized.

## 14. Generator expressions

Python also supports generator expressions.

A list comprehension creates a list:

```python
ids = [f"deployment-{n}" for n in range(1, 4)]
```

A generator expression creates an iterator-like generator object:

```python
ids = (f"deployment-{n}" for n in range(1, 4))
```

The values can then be consumed incrementally:

```python
for deployment_id in ids:
    print(deployment_id)
```

Generator expressions are useful when the resulting sequence is intended to be consumed rather than retained as a list.

## 15. `yield from`

`yield from` delegates iteration to another iterable.

```python
def validation_steps():
    yield "syntax-check"
    yield "config-check"


def deployment_pipeline():
    yield from validation_steps()
    yield "deploy"
```

Now:

```python
for step in deployment_pipeline():
    print(step)
```

produces the values from `validation_steps()` followed by `deploy`.

The official semantics of `yield from` are more extensive than simple looping: it delegates generator control to a subiterator and participates in the generator protocol, including `send()`, `throw()`, and completion behavior.

For normal DevOps automation, the basic delegation pattern is the important starting point. The full delegation protocol can be studied later when advanced generator control is required.

## 16. Returning a value from a generator

A generator function can use `return` to finish iteration.

```python
def deployment():
    yield "deploying"
    return "deployment complete"
```

The generator finishes by raising `StopIteration`. The returned value becomes the `value` associated with that `StopIteration` instance.

This is most useful when working directly with generator protocol behavior or `yield from`. Ordinary `for` loops consume the yielded values and do not expose the return value as a normal loop item.

## 17. Generator methods: advanced control

Generator iterators provide methods beyond `__next__()`, including mechanisms for sending values into a suspended generator, throwing exceptions into it, and closing it.

The main methods are:

- `__next__()` / `next()` — resume execution normally;
- `send(value)` — resume execution and make the current `yield` expression evaluate to the supplied value;
- `throw(...)` — raise an exception at the suspended `yield` point;
- `close()` — request that the generator terminate.

Example using `send()`:

```python
def receiver():
    value = yield "ready"
    yield f"received: {value}"

worker = receiver()

print(next(worker))
print(worker.send("deployment-42"))
```

This is an advanced generator feature. Most automation code should begin with ordinary `yield` and iteration before introducing bidirectional generator communication.

## 18. Iteration and files

Iteration is especially useful for processing files incrementally.

```python
with open("deployment.log") as log_file:
    for line in log_file:
        process(line)
```

The file object supports iteration, so the code can process each line through the iteration interface rather than first constructing a separate list of all lines.

This is a useful pattern for logs, configuration data, reports, and other potentially large inputs.

The exact resource-management behavior is a separate concern: use `with` so the file is managed explicitly rather than relying on garbage-collection timing.

## 19. Iteration and command output

When automation code receives a collection of command results, an iterator or generator can make the processing pipeline easier to compose.

For example:

```python
def successful(results):
    for result in results:
        if result.returncode == 0:
            yield result
```

A consumer can then process only successful results:

```python
for result in successful(results):
    print(result)
```

This is a **repository teaching example**. It does not prescribe a particular command-execution library or result object.

## 20. Iterator pipelines

Generators can be chained together to form a processing pipeline.

```python
def non_empty(lines):
    for line in lines:
        if line.strip():
            yield line


def errors(lines):
    for line in lines:
        if "ERROR" in line:
            yield line


pipeline = errors(non_empty(lines))

for line in pipeline:
    print(line)
```

The pipeline expresses the transformation stages separately.

A useful pattern is:

```text
input
  ↓
filter
  ↓
transform
  ↓
filter
  ↓
consumer
```

This can make automation code easier to compose because each stage can consume an iterable and produce another iterable.

## 21. Generator pipelines are lazy

Consider:

```python
def numbers():
    for number in range(1, 4):
        print(f"producing {number}")
        yield number

values = numbers()
```

At this point, the generator has been created but its body has not been run through all iterations.

The work happens as values are requested:

```python
for value in values:
    print(f"consuming {value}")
```

This is called **lazy evaluation** in the context of generator-based iteration: values are produced when iteration advances rather than all being produced at generator creation time.

## 22. A generator can represent an unbounded stream

Iterators do not have to be finite.

For example:

```python
def sequence():
    number = 0
    while True:
        yield number
        number += 1
```

This generator has no natural endpoint.

A consumer must decide when to stop:

```python
for number in sequence():
    print(number)
    if number == 5:
        break
```

This pattern can be useful for conceptualizing streams, polling sequences, or generated work. In real automation, an unbounded loop must also account for cancellation, timeouts, backoff, and resource usage where applicable.

## 23. `itertools`

The standard library includes `itertools`, which provides building blocks for iterator-based processing.

Typical categories include:

- infinite iterator building blocks;
- iterator combinations;
- filtering;
- grouping;
- chaining;
- slicing and selection.

For example, `itertools.chain()` can combine multiple iterables into a single iteration stream:

```python
from itertools import chain

for item in chain(["api", "worker"], ["scheduler"]):
    print(item)
```

Use the official `itertools` documentation when selecting a specific function. Do not memorize a large collection of utilities without understanding the iterator protocol first.

## 24. Iterator exhaustion is a common automation bug

A subtle bug can occur when a function consumes an iterator and another part of the program expects to iterate over it again.

For example:

```python
records = (record for record in source)

first_pass = list(records)
second_pass = list(records)
```

`second_pass` is empty because the generator has already been consumed.

If multiple independent passes are required, keep a reusable iterable or materialize the data deliberately when that is appropriate.

The correct design depends on the data size and whether repeated traversal is actually required.

## 25. Iterable versus iterator: a practical test

When learning a new Python object, ask two questions:

1. Can I call `iter(obj)` successfully?
2. Does the returned object produce values with `next()`?

For an iterator, this relationship holds:

```python
iterator = iter(obj)
assert iter(iterator) is iterator
```

The second property is part of the iterator protocol.

The fact that an object is iterable does not by itself mean that it is its own iterator.

## 26. Common mistakes

### Mistake 1: Assuming every iterable is an iterator

A list is iterable, but a list is not itself the iterator returned by `iter(list_object)`.

### Mistake 2: Reusing an exhausted iterator

Once an iterator is exhausted, ordinary iteration over that iterator will not restart it.

### Mistake 3: Converting every generator to a list immediately

If the point of the pipeline is incremental processing, immediately materializing the generator defeats that design.

Materialize intentionally when you actually need a collection or multiple passes.

### Mistake 4: Assuming generators automatically make code faster

Generators change evaluation and storage behavior. They do not guarantee better performance for every workload.

### Mistake 5: Hiding expensive work behind a generator

Lazy execution can make when work happens less obvious. Document or structure generator pipelines so that callers understand when external work occurs.

### Mistake 6: Building an infinite generator without a stopping condition

An unbounded generator is valid, but the consumer needs an explicit stopping strategy.

### Mistake 7: Manually implementing iteration when a generator is clearer

A custom iterator class is appropriate when explicit iterator state and object behavior are useful. Otherwise, a generator can often express the same iteration logic more directly.

## 27. DevOps design patterns

### Pattern 1 — Process large input incrementally

```python
def relevant_lines(lines):
    for line in lines:
        if "ERROR" in line:
            yield line
```

Use this when downstream processing can operate one record at a time.

### Pattern 2 — Separate filtering from consumption

```python
def production_targets(targets):
    for target in targets:
        if target.environment == "production":
            yield target
```

The caller decides what to do with the selected targets.

### Pattern 3 — Build small pipeline stages

```python
def normalize(names):
    for name in names:
        yield name.strip().lower()


def non_empty(names):
    for name in names:
        if name:
            yield name
```

Then compose them:

```python
pipeline = non_empty(normalize(names))
```

### Pattern 4 — Keep external side effects at the consumer boundary

A generator can prepare or transform data, while the final consumer performs the deployment, API mutation, or other external side effect.

This separation is a **repository design recommendation**, not a Python language requirement.

## 28. Practical exercises

### Exercise 1 — Manual iteration

Create a list of deployment targets. Use `iter()` and repeated `next()` calls to consume it. Handle `StopIteration` explicitly.

### Exercise 2 — Custom iterable

Create a `DeploymentTargets` class whose `__iter__()` method returns an iterator over its internal collection.

### Exercise 3 — Custom iterator

Implement a countdown iterator with `__iter__()` and `__next__()`. Make sure it raises `StopIteration` at the correct point.

### Exercise 4 — Generator

Write a generator that yields deployment stages one at a time.

### Exercise 5 — Generator pipeline

Build two generator functions: one filters records and one transforms them. Chain them together and consume the final result.

### Exercise 6 — File processing

Iterate over a log file and yield only lines that satisfy a condition. Do not first read the entire file into a list.

### Exercise 7 — Exhaustion

Create a generator, consume it once, then attempt to consume it again. Explain the result.

### Exercise 8 — `yield from`

Create two generators representing separate phases of an automation workflow. Use `yield from` to combine their output into a single generator.

## 29. Knowledge checks

Before moving on, you should be able to answer:

1. What is an iterable?
2. What is an iterator?
3. What is the difference between an iterable and an iterator?
4. What do `iter()` and `next()` do?
5. What is the iterator protocol?
6. Why does `StopIteration` exist?
7. Why are iterators generally consumable?
8. What happens when a generator function is called?
9. What does `yield` do to generator execution?
10. Why can generator functions retain local state between values?
11. What is the difference between a list comprehension and a generator expression?
12. What does `yield from` do?
13. Why can generators be useful for large or streaming inputs?
14. Why does a generator not automatically guarantee better performance?
15. When would you choose a custom iterator class instead of a generator?
16. Why can an iterator pipeline be useful in DevOps automation?

## 30. Completion checklist

You are ready to continue when you can:

- explain iterable versus iterator clearly;
- use `iter()` and `next()` manually;
- explain `StopIteration`;
- describe what a `for` loop is doing at the iterator-protocol level;
- implement a simple custom iterable;
- implement a simple custom iterator;
- write generator functions with `yield`;
- explain generator suspension and resumption;
- use generator expressions appropriately;
- use `yield from` for basic delegation;
- recognize iterator exhaustion;
- build a simple lazy processing pipeline;
- decide when materializing a collection is appropriate;
- apply iterator and generator patterns to automation-oriented data processing.

## 31. Verified facts vs repository design

### Verified facts

The official Python documentation establishes the iterator protocol, the behavior of `iter()`/`next()`, `StopIteration`, generator functions, generator iterators, `yield`, and `yield from`.

### Repository design decisions

This guide emphasizes incremental processing, explicit data flow, small pipeline stages, and separation of data preparation from external side effects because these are useful teaching patterns for DevOps automation.

These recommendations are not Python language requirements.

### Demonstrations

Examples using deployment names, logs, command results, and pipeline stages are teaching demonstrations created for this repository. They do not claim that any particular DevOps product exposes those exact objects or interfaces.

## 32. Authoritative references

- Python — Built-in Types, Iteration-related types and Generator types:
  https://docs.python.org/3/library/stdtypes.html#iterator-types
- Python — Data model, generator functions:
  https://docs.python.org/3/reference/datamodel.html#generator-functions
- Python — Expressions, yield expressions and generator expressions:
  https://docs.python.org/3/reference/expressions.html#yield-expressions
  
  https://docs.python.org/3/reference/expressions.html#generator-expressions
- Python — Functional Programming HOWTO, Iterators and generators:
  https://docs.python.org/3/howto/functional.html
- Python — `itertools`:
  https://docs.python.org/3/library/itertools.html

For version-specific behavior, consult the Python documentation for the interpreter version being used.