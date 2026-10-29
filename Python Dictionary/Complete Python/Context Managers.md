# Context Managers

Context managers provide a structured way to run setup and cleanup around a block of code. They are especially important in automation because resources such as files, connections, locks, and temporary state often need deterministic cleanup.

This topic builds on **Decorators**, **Iterators and Generators**, and **Scopes and Python Data Model**.

> **Source note:** The language behavior in this file follows the official Python documentation. Examples labeled as demonstrations are repository teaching examples, not claims about an external system.

## 1. The core mental model

A context manager defines what should happen when entering and leaving a managed block.

```text
enter resource/context
        │
        ▼
   with block
        │
        ▼
exit resource/context
```

The central syntax is:

```python
with expression as value:
    # work inside the managed context
    ...
```

The `with` statement is designed for initialization and finalization around a block of code.

## 2. Why context managers matter

Consider a file:

```python
file = open("config.txt")
try:
    contents = file.read()
finally:
    file.close()
```

A context manager lets the resource-management boundary be expressed directly:

```python
with open("config.txt") as file:
    contents = file.read()
```

The important idea is not merely shorter syntax. The context manager makes the resource lifetime explicit and ties cleanup to leaving the block.

Python's data-model documentation recommends explicit resource management for objects that hold external resources and identifies `with` and `try`/`finally` as convenient mechanisms for cleanup.

## 3. The `with` statement

The `with` statement evaluates a context expression and uses the context manager protocol to enter and leave the managed block.

A useful conceptual model is:

```text
1. evaluate context expression
2. enter the context
3. run the with-block
4. leave the context
```

The exact exception-handling behavior is part of Python's language specification; do not replace it with an informal rule such as “the cleanup always succeeds.” Cleanup code can itself raise exceptions.

## 4. `__enter__` and `__exit__`

A class can implement the context manager protocol with `__enter__()` and `__exit__()`.

```python
class ManagedOperation:
    def __enter__(self):
        print("enter")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("exit")
```

It can then be used as:

```python
with ManagedOperation() as operation:
    print("working")
```

The value returned by `__enter__()` is what is assigned to the target after `as`, when one is present.

## 5. The `as` target

These two pieces have different responsibilities:

```python
with ManagedOperation() as operation:
    ...
```

- `ManagedOperation()` produces the context manager object.
- `__enter__()` produces the value assigned to `operation`.

The two objects do not have to be conceptually identical.

For example, a context manager may manage a resource internally while returning a separate object that the block should use.

## 6. `__exit__` and exceptions

`__exit__()` receives information about an exception if one occurred while executing the managed block.

Its parameters are conventionally written as:

```python
def __exit__(self, exc_type, exc_value, traceback):
    ...
```

If no exception occurred, the exception-related arguments indicate normal completion.

A context manager can use this information to perform cleanup, logging, or other policy-specific handling.

The return value of `__exit__()` is significant: returning a true value indicates that the exception should be suppressed. Returning a false value, including the usual `None`, allows the exception to propagate.

### Important engineering rule

Do not suppress exceptions accidentally.

A context manager used for infrastructure automation should have a clear contract about which failures, if any, it intentionally handles.

## 7. Cleanup with `try`/`finally`

A context manager is closely related to the resource-management pattern:

```python
resource = acquire()
try:
    use(resource)
finally:
    release(resource)
```

The `finally` block expresses cleanup that should be attempted when leaving the protected code.

Context managers package this pattern into a reusable abstraction.

## 8. A custom resource manager

Teaching example:

```python
class Resource:
    def connect(self):
        print("connected")

    def close(self):
        print("closed")


class ManagedResource:
    def __enter__(self):
        self.resource = Resource()
        self.resource.connect()
        return self.resource

    def __exit__(self, exc_type, exc_value, traceback):
        self.resource.close()
```

Usage:

```python
with ManagedResource() as resource:
    print("using resource")
```

This demonstrates the lifecycle pattern without depending on a particular external service.

## 9. `contextlib.contextmanager`

The standard library's `contextlib` module provides `contextmanager()` for creating context managers with a generator function.

```python
from contextlib import contextmanager


@contextmanager
def managed_resource():
    resource = acquire_resource()
    try:
        yield resource
    finally:
        release_resource(resource)
```

The code before `yield` represents setup, the yielded value is available to the `with` block, and the code after `yield` is resumed when the block exits.

The official documentation specifies that the decorated function must produce a generator-iterator that yields exactly one value.

## 10. Why `try`/`finally` matters with `@contextmanager`

A common pattern is:

```python
@contextmanager
def managed_resource():
    resource = acquire_resource()
    try:
        yield resource
    finally:
        release_resource(resource)
```

The `finally` block is important because the body of the `with` statement may:

- complete normally;
- raise an exception;
- return from a function;
- otherwise leave the managed block.

The cleanup policy should therefore be expressed deliberately rather than relying on a later garbage-collection event.

## 11. Exceptions inside a generator-based context manager

If an unhandled exception occurs inside the `with` block, the exception is reraised inside the generator at the point where it yielded.

That makes this structure useful:

```python
@contextmanager
def managed_resource():
    resource = acquire_resource()
    try:
        yield resource
    except Exception:
        record_failure()
        raise
    finally:
        release_resource(resource)
```

The `raise` is important when the intention is to record the failure but still propagate it.

Silently swallowing a deployment or automation failure can produce a false-success result.

## 12. Context managers and files

File objects are a common practical example of context management:

```python
with open("inventory.txt") as file:
    for line in file:
        process(line)
```

The block defines the period during which the file is being used.

This pattern is preferable to relying on object finalization to close external resources.

## 13. Context managers and DevOps automation

Context managers are useful whenever a piece of automation has a clear lifecycle.

Examples include teaching patterns for:

- opening and closing files;
- acquiring and releasing locks;
- starting and cleaning up temporary resources;
- temporarily changing process state;
- managing test fixtures;
- opening and closing connections;
- recording setup and teardown around an operation.

Whether a particular external client supports context management must be verified from that client's authoritative documentation. Do not assume that every connection, SDK object, or API client implements the protocol.

## 14. `contextlib` utilities

`contextlib` contains several standard utilities for context management.

### `contextmanager`

Creates a context manager from a generator function.

### `closing`

Creates a context manager that calls an object's `close()` method when the block finishes. It is primarily useful for objects that do not already provide native context-manager support.

### `nullcontext`

Provides a context manager that can act as a no-op context when an operation is optional.

### `suppress`

Suppresses specified exceptions inside a `with` block. It should be used only when deliberately ignoring those specific failures is correct.

### `redirect_stdout` and `redirect_stderr`

Temporarily redirect Python's standard output or error streams. The standard documentation notes that these change global process state and therefore have limitations in library and concurrent code.

### `chdir`

Temporarily changes the current working directory and restores it when the context exits. The official documentation warns that changing this global state is not suitable for most threaded or asynchronous contexts.

## 15. Context managers and decorators

Context managers and decorators solve different problems, but Python connects them through `contextlib.ContextDecorator`.

A context manager can be made usable as a decorator:

```python
from contextlib import ContextDecorator


class OperationContext(ContextDecorator):
    def __enter__(self):
        print("start")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("finish")
        return False
```

This lets one lifecycle policy be expressed around a `with` block or around a function call.

Use this only when the resulting control flow remains easy to understand.

## 16. Asynchronous context managers

Python also supports asynchronous context managers with `async with`.

The corresponding protocol uses `__aenter__()` and `__aexit__()`.

The standard library also provides `contextlib.asynccontextmanager()` for creating asynchronous context managers from asynchronous generator functions.

Example:

```python
from contextlib import asynccontextmanager


@asynccontextmanager
async def managed_connection():
    connection = await acquire_connection()
    try:
        yield connection
    finally:
        await release_connection(connection)
```

The details of the underlying asynchronous resource still depend on the resource implementation.

## 17. Multiple context managers

A `with` statement can manage multiple contexts:

```python
with first_context() as first, second_context() as second:
    use(first, second)
```

This is useful when multiple resources have related lifetimes.

Keep the statement readable. If setup becomes complicated, separate context managers or a dedicated abstraction may be clearer.

## 18. Context managers and idempotent cleanup

In infrastructure automation, cleanup operations should be designed carefully.

A repository teaching example might model cleanup as:

```python
@contextmanager
def temporary_workspace():
    path = create_workspace()
    try:
        yield path
    finally:
        remove_workspace(path)
```

Whether `remove_workspace()` is safe to call after partial setup or after an earlier failure is an application-specific property.

Do not assume cleanup operations are idempotent merely because they occur in `finally`.

## 19. Common mistakes

### Mistake 1: Relying on garbage collection for external resources

External resources should have explicit lifecycle management where the API provides it.

### Mistake 2: Suppressing every exception

Avoid:

```python
def __exit__(self, exc_type, exc_value, traceback):
    return True
```

unless suppressing all exceptions is explicitly part of the contract.

### Mistake 3: Logging an exception and forgetting to re-raise

If the intended behavior is reporting rather than suppression, re-raise the exception.

### Mistake 4: Cleanup that can hide the original failure

Cleanup code can itself fail. Production designs should decide how cleanup failures interact with the original operation failure.

### Mistake 5: Assuming every object is a context manager

Verify the external API before writing:

```python
with some_client() as client:
    ...
```

### Mistake 6: Changing global state inside concurrent code

Utilities such as changing the current working directory or redirecting standard streams have global effects and require careful concurrency considerations.

## 20. Practical exercises

### Exercise 1 — File lifecycle

Write a small program that reads a configuration file with `with` and explain what resource-management problem the syntax solves.

### Exercise 2 — Custom context manager

Create a class with `__enter__()` and `__exit__()` that records when an operation starts and finishes.

Test both:

- normal completion;
- an exception inside the block.

### Exercise 3 — Exception policy

Modify the context manager so it records an exception and then allows the exception to propagate. Explain why returning `False` or `None` differs from returning `True`.

### Exercise 4 — `@contextmanager`

Rewrite your class-based context manager using `contextlib.contextmanager`.

Use `try`/`finally` for cleanup.

### Exercise 5 — Temporary automation state

Design a context manager that temporarily changes a piece of application state and restores the previous value on exit.

Document why restoration is required if the operation inside the block fails.

### Exercise 6 — DevOps lifecycle design

Choose one automation resource—temporary workspace, lock, file, connection, or test fixture—and design its setup, normal-use, failure, and cleanup paths before implementing it.

## 21. Knowledge checks

1. What problem does a context manager solve?
2. What roles do `__enter__()` and `__exit__()` play?
3. What value is assigned by `with resource() as value:`?
4. What does a truthy return value from `__exit__()` mean?
5. Why is `try`/`finally` important for resource cleanup?
6. How does `@contextmanager` turn a generator function into a context manager?
7. Why should a context manager avoid accidentally suppressing exceptions?
8. When would `nullcontext()` be useful?
9. Why can changing the working directory be dangerous in concurrent code?
10. What is the difference between a synchronous and asynchronous context manager?

## 22. Completion checklist

You should be able to:

- explain the purpose of a context manager;
- use `with` for resource lifecycles;
- implement `__enter__()` and `__exit__()`;
- explain exception handling in `__exit__()`;
- use `contextlib.contextmanager`;
- write cleanup with `try`/`finally`;
- distinguish exception reporting from exception suppression;
- recognize when global state makes a context manager unsafe for concurrent use;
- explain the difference between `with` and `async with`;
- design explicit setup and cleanup boundaries for automation code.

## 23. Verified facts vs repository design

### Verified facts

- Python's `with` statement provides a mechanism for initialization and finalization around a block.
- Context managers implement the synchronous context-manager protocol through `__enter__()` and `__exit__()`.
- The value returned by `__enter__()` can be bound by the `as` target.
- A truthy result from `__exit__()` suppresses an exception raised in the managed block.
- `contextlib.contextmanager` provides a generator-based way to define context managers.
- Python supports asynchronous context managers through `async with`, `__aenter__()`, and `__aexit__()`.

### Repository design decisions

- DevOps examples focus on resource lifecycles, cleanup, failure handling, and explicit automation boundaries.
- Examples avoid assuming that third-party SDKs or infrastructure clients implement context management.
- Exercises intentionally require learners to define cleanup and exception policy before implementation.

### Demonstrations

The custom classes, logging examples, and automation-resource examples in this file are teaching demonstrations created for this repository. They are not official Python examples and do not represent any external product's API.

## 24. Authoritative references

- Python `with` statement: https://docs.python.org/3/reference/compound_stmts.html#the-with-statement
- Python data model — context managers: https://docs.python.org/3/reference/datamodel.html#context-managers
- `contextlib` — Utilities for `with` statements: https://docs.python.org/3/library/contextlib.html
- Python language reference: https://docs.python.org/3/reference/
