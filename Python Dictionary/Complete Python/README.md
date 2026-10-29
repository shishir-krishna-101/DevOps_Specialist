# Complete Python

A deeper continuation of **Python for DevOps**.

This track is for learners who already understand the Python fundamentals needed for scripting and automation and want to become more complete Python developers. It deliberately does **not** repeat the beginner material already covered in [Python for DevOps](../Python%20for%20DevOps/README.md).

> **Learning rule:** you do not need to finish every topic here before doing DevOps work. Many of these topics become valuable when you build larger tools, libraries, services, test suites, or automation systems.

## What this track adds

The DevOps-focused path teaches enough Python to build useful scripts and automation. This continuation goes deeper into:

- object-oriented design and the Python data model
- advanced functions, closures, decorators, and context managers
- iterators, generators, and lazy computation
- modules, packages, imports, and application structure
- dataclasses and stronger type annotations
- functional-programming tools in the standard library
- advanced testing and mocking
- concurrency and asynchronous programming
- databases with SQLite
- debugging, profiling, and performance investigation
- packaging and reusable Python projects
- deeper exception, resource-management, and API-design techniques
- selected advanced language features

---

## 1. How to use this continuation

### Recommended prerequisite

Before starting, you should be comfortable with:

- variables and common built-in types
- conditions and loops
- lists, dictionaries, sets, and tuples
- functions
- modules and imports
- exceptions
- files and `pathlib`
- JSON and command-line programs
- basic testing
- virtual environments

These are already introduced in the DevOps-focused track.

### Suggested progression

```text
Python for DevOps
        ↓
Functions and Python data model
        ↓
OOP and reusable design
        ↓
Iterators / generators / decorators / context managers
        ↓
Modules / packages / typing / dataclasses
        ↓
Testing and mocking
        ↓
Concurrency and asyncio
        ↓
Databases / networking / application structure
        ↓
Packaging / performance / debugging
        ↓
Advanced Python topics
```

Do not treat this as a race. Build small programs after each major section.

---

# 2. Advanced functions

Python functions are objects, which means they can be stored, passed to other functions, and returned from functions.

## 2.1 Positional-only and keyword-only parameters

Python supports different parameter styles. Understanding them helps you design clear APIs.

```python
def connect(host, /, port=443, *, timeout=10):
    return host, port, timeout
```

Here:

- `host` is positional-only.
- `port` can be positional or keyword-based.
- `timeout` is keyword-only.

Use these features when an API benefits from making argument intent explicit.

## 2.2 First-class functions

```python
def square(value):
    return value * value

operation = square
print(operation(5))
```

A function can be treated as a value.

This idea is the foundation for callbacks, decorators, higher-order functions, and many functional-programming patterns.

## 2.3 Closures

A nested function can retain access to variables from its enclosing scope.

```python
def make_multiplier(factor):
    def multiply(value):
        return value * factor
    return multiply

triple = make_multiplier(3)
print(triple(10))
```

The returned function retains the value of `factor`.

### Learn next

- lexical scope
- `nonlocal`
- nested functions
- closures versus objects
- when a closure improves readability and when a class is clearer

---

# 3. Object-oriented Python

Python classes bundle data and functionality into objects. The official Python tutorial covers classes, instance objects, methods, inheritance, namespaces, and multiple inheritance.

## 3.1 Class and instance

```python
class Server:
    def __init__(self, hostname, environment):
        self.hostname = hostname
        self.environment = environment

    def describe(self):
        return f"{self.hostname} ({self.environment})"

server = Server("app-01", "production")
print(server.describe())
```

Think of:

- **class** = definition of a type
- **instance** = one object created from that type
- **attribute** = state stored on an object
- **method** = behavior associated with the object

## 3.2 Instance variables versus class variables

```python
class Service:
    category = "service"

    def __init__(self, name):
        self.name = name
```

`category` is defined on the class, while `name` is assigned for each instance.

Be careful with mutable class attributes because they can be shared by instances.

## 3.3 Inheritance

Inheritance lets one class derive behavior from another.

```python
class Resource:
    def describe(self):
        return "resource"

class Server(Resource):
    def describe(self):
        return "server"
```

Learn inheritance, method overriding, and `super()`.

### Prefer composition when it models the problem better

Composition means building an object from other objects instead of creating a large inheritance hierarchy.

```python
class Logger:
    def write(self, message):
        print(message)

class HealthChecker:
    def __init__(self, logger):
        self.logger = logger
```

For maintainable automation tools, smaller cooperating objects are often easier to test than deep inheritance trees.

## 3.4 Properties

Properties allow attribute-style access while controlling how a value is calculated or changed.

```python
class Service:
    def __init__(self, name):
        self._name = name

    @property
    def name(self):
        return self._name
```

Learn setters only when validation or controlled mutation genuinely requires them.

## 3.5 Special methods

Python objects can participate in language operations through special methods such as:

- `__init__`
- `__repr__`
- `__str__`
- `__eq__`
- `__len__`
- `__iter__`
- `__enter__` / `__exit__`

These methods connect your classes to Python's object model.

## Exercises

1. Create a `Deployment` class with validated state.
2. Add a useful `__repr__`.
3. Implement equality for two deployment objects.
4. Build the same small model once with inheritance and once with composition. Compare the designs.

---

# 4. Python namespaces, scopes, and the data model

Understanding names and objects explains many Python behaviors that otherwise look surprising.

Study:

- local, enclosing, global, and built-in scopes
- name binding
- object identity versus equality
- mutable versus immutable objects
- attribute lookup
- special methods
- how operators map to object behavior

Example:

```python
a = [1, 2]
b = a
b.append(3)

print(a)  # [1, 2, 3]
```

`a` and `b` refer to the same list object.

Do not use `is` as a general replacement for `==`. Identity and equality answer different questions.

### Useful mental model

```text
name ───────► object
               │
               ├── type
               ├── value/state
               └── behavior
```

---

# 5. Iterators and generators

The `for` loop uses the iterator protocol. An iterator supplies values one at a time and signals completion with `StopIteration`.

## 5.1 Iterator protocol

```python
items = iter(["a", "b", "c"])

print(next(items))
print(next(items))
```

A custom iterator normally provides `__iter__()` and `__next__()`.

## 5.2 Generators

Generators provide a compact way to create iterators using `yield`.

```python
def read_names(lines):
    for line in lines:
        name = line.strip()
        if name:
            yield name
```

A generator pauses at `yield` and resumes when another value is requested.

This is useful when processing large streams of data without first constructing the entire result in memory.

## 5.3 Generator expressions

```python
values = (number * 2 for number in range(10))
```

Compare this with a list comprehension when deciding whether you need all results immediately or want lazy iteration.

## 5.4 `yield from`

`yield from` delegates iteration to another iterable or generator.

Learn it after ordinary generators are comfortable.

### Practical exercises

- stream a large log file line by line
- create a generator that yields failed records
- combine multiple generators
- compare memory behavior of a list and generator for the same logical data

---

# 6. Decorators

A decorator wraps or transforms a callable.

```python
from functools import wraps


def announce(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"running {func.__name__}")
        return func(*args, **kwargs)

    return wrapper


@announce
def deploy(environment):
    return f"deploying to {environment}"
```

Learn:

- functions as values
- nested functions
- `*args` and `**kwargs`
- `functools.wraps`
- decorators with arguments
- stacking decorators

### DevOps applications

Decorators can be useful for cross-cutting behavior such as instrumentation or controlled retries, but avoid hiding important operational behavior behind too much abstraction.

---

# 7. Context managers

A context manager controls setup and cleanup around a block of code.

```python
with open("config.txt") as file:
    data = file.read()
```

The `with` statement is especially important for reliable resource management.

## 7.1 Custom context manager

```python
class ManagedResource:
    def __enter__(self):
        print("acquire")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("release")
        return False
```

Also learn the `contextlib` module for utilities that can make context-manager implementations simpler.

### Practice

Build a context manager that:

1. records the start time,
2. runs a block of code,
3. records completion even when the block raises an exception.

---

# 8. Comprehensions and functional tools

Comprehensions are concise ways to construct collections.

```python
healthy = [host for host in hosts if host.status == "healthy"]
```

Do not sacrifice readability for a one-line expression.

## 8.1 `itertools`

The standard library's `itertools` module provides iterator-building tools for efficient looping.

Learn progressively:

- `chain`
- `islice`
- `repeat`
- `product`
- `permutations`
- `combinations`
- grouping and filtering tools

Use the official documentation to understand the exact behavior of each function before using it in production code.

## 8.2 `functools`

The `functools` module provides higher-order-function utilities.

Important tools include:

- `partial`
- `wraps`
- `reduce`
- caching utilities
- comparison helpers

Use `reduce` only when it makes the transformation clearer than an explicit loop.

---

# 9. Useful advanced collections

The `collections` module provides specialized containers.

Study:

- `Counter` for counting
- `defaultdict` for default values
- `deque` for efficient operations at both ends
- `ChainMap` for layered mappings
- `namedtuple` and modern alternatives where appropriate

Example:

```python
from collections import Counter

statuses = ["ok", "failed", "ok", "ok"]
counts = Counter(statuses)
print(counts["ok"])
```

Choose the data structure based on the operations your program actually needs.

---

# 10. Dataclasses

`dataclasses` can generate common methods for classes based on annotated fields.

```python
from dataclasses import dataclass


@dataclass
class Server:
    hostname: str
    environment: str
    port: int = 443
```

Dataclasses are useful for data-oriented models where writing boilerplate methods manually adds little value.

Learn:

- `@dataclass`
- default values
- `field()`
- immutable/frozen models
- ordering and equality options
- keyword-only fields where appropriate
- nested dataclasses

Do not assume that every class should be a dataclass. Classes with substantial behavior may need a different design.

---

# 11. Type hints and the `typing` module

Python annotations describe intended types, but Python does not generally enforce function and variable annotations at runtime. Type checkers, IDEs, and other tools can use them.

```python
def restart(service: str, attempts: int) -> bool:
    ...
```

Study:

- basic annotations
- `list[str]`, `dict[str, int]`, and other generic forms
- unions
- `Callable`
- `TypeVar`
- `Protocol`
- `TypedDict`
- `Literal`
- `Final`
- type aliases
- annotations on classes and methods

### Why this matters

Type hints can make larger automation codebases easier to navigate and review. They are particularly useful when functions pass structured data between many modules.

### Important distinction

```text
Type annotation
      ↓
information for humans + development tools
      ↓
not automatic runtime validation
```

If runtime validation is required, design that validation explicitly.

---

# 12. Modules, packages, and imports

As projects grow, putting everything into one script becomes difficult to maintain.

Learn:

- module names
- package structure
- absolute imports
- relative imports
- module search path
- `__name__`
- `__main__`
- package initialization
- import-time side effects
- circular-import problems

## 12.1 Executable module pattern

```python
def main():
    print("application entry point")


if __name__ == "__main__":
    main()
```

This allows a module to be imported without automatically executing its command-line entry behavior.

## 12.2 A maintainable project shape

```text
my_tool/
├── pyproject.toml
├── src/
│   └── my_tool/
│       ├── __init__.py
│       ├── cli.py
│       ├── config.py
│       └── operations.py
└── tests/
    ├── test_config.py
    └── test_operations.py
```

This is a **repository design example**, not a statement that every Python project must use this exact structure.

---

# 13. Packaging and reusable Python projects

Once a script becomes a reusable application or library, learn Python packaging rather than continuing to treat the project as a single file.

Study:

- project metadata
- `pyproject.toml`
- build systems
- package dependencies
- editable development installs
- building distributions
- publishing considerations
- versioning
- console entry points

Use the official Python Packaging User Guide for current packaging standards and tooling rather than relying on old tutorials.

Source: https://packaging.python.org/en/latest/

### Practice project

Turn one of your earlier DevOps scripts into a small installable Python project with:

- package code
- tests
- project metadata
- a command-line entry point
- documentation

---

# 14. Advanced testing

The `unittest` framework includes tools for replacing dependencies with mocks during tests.

```python
from unittest.mock import Mock

client = Mock()
client.get_status.return_value = "healthy"

assert client.get_status() == "healthy"
```

Learn:

- `Mock`
- `MagicMock`
- `patch`
- `side_effect`
- `return_value`
- call assertions
- `AsyncMock`
- testing failure paths
- testing code that calls external systems

## Mock the boundary, not everything

If a function talks to a cloud API, filesystem, subprocess, or HTTP client, isolate that external boundary when unit testing.

Then use integration tests for behavior that actually requires the external system.

### Exercise

Take a health-check program and test:

1. healthy response
2. unhealthy response
3. timeout/error
4. malformed response
5. retry behavior

---

# 15. Concurrency

Concurrency becomes important when a program must handle multiple independent tasks efficiently.

Python provides several different concurrency approaches. They are not interchangeable.

## 15.1 Threads

Use the `threading` ecosystem when thread-based concurrency fits the workload and libraries involved.

Typical learning examples include multiple independent I/O operations.

## 15.2 Processes

`multiprocessing` can run work in separate processes. It is useful for workloads where process-based execution is appropriate.

## 15.3 `concurrent.futures`

`concurrent.futures` provides a higher-level interface for asynchronously executing callables, including thread and process executors.

```python
from concurrent.futures import ThreadPoolExecutor


def check(host):
    return host, "ok"

with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(check, hosts))
```

The number of workers should be chosen based on the workload and tested rather than copied blindly.

---

# 16. Asynchronous Python with `asyncio`

`asyncio` provides asynchronous programming using `async` and `await`. The official documentation describes it as particularly useful for I/O-bound and high-level network code.

Basic example:

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

Learn in this order:

1. coroutine functions
2. `await`
3. tasks
4. concurrent coroutines
5. cancellation
6. timeouts
7. synchronization primitives
8. async context managers
9. async iterators
10. integration with real network clients

### Do not confuse these models

```text
threading / executors  → thread-based concurrency
multiprocessing        → process-based concurrency
asyncio                → asynchronous coroutines and event-loop based execution
```

Choose based on the workload and the APIs you are using.

---

# 17. SQLite and database programming

Python includes `sqlite3`, an interface to SQLite databases.

SQLite is useful for learning database programming and for applications where an embedded database is appropriate.

Basic flow:

```python
import sqlite3

connection = sqlite3.connect("inventory.db")

try:
    cursor = connection.cursor()
    cursor.execute(
        "CREATE TABLE IF NOT EXISTS hosts (name TEXT, status TEXT)"
    )
    connection.commit()
finally:
    connection.close()
```

Learn:

- connections
- cursors
- SQL statements
- parameters
- transactions
- commits and rollbacks
- context management
- query results
- schema design basics

Do not build SQL by concatenating untrusted input. Learn parameterized queries and transaction boundaries.

---

# 18. Serialization and data formats

Beyond JSON, learn the differences between:

- text serialization
- structured data formats
- binary serialization
- application-specific schemas

Be especially careful with Python-specific serialization mechanisms such as `pickle` when handling untrusted data. Read the official security guidance before using them across trust boundaries.

For portable automation interfaces, prefer formats and schemas that are appropriate for the systems communicating with each other.

---

# 19. Networking with the standard library

For deeper Python development, understand the standard networking building blocks:

- `urllib`
- `socket`
- `ssl`
- `selectors`
- `ipaddress`

You do not need to use low-level sockets for every automation task. Learn them to understand what higher-level networking libraries are building upon.

### Practice

Build a small TCP client/server experiment on your local machine and document:

- connection lifecycle
- encoding/decoding
- timeouts
- failure behavior
- resource cleanup

---

# 20. Regular expressions — deeper level

If the DevOps path introduced basic pattern matching, continue with:

- groups
- named groups
- alternation
- lookarounds
- compiled patterns
- flags
- substitution
- match objects
- performance considerations

Use regular expressions when they improve the solution. For structured formats such as JSON, do not replace a parser with a regular expression.

---

# 21. Dates and time zones

Learn the standard-library `datetime` model in depth:

- date versus time
- naive versus aware datetimes
- durations with `timedelta`
- time-zone-aware operations
- formatting and parsing
- timestamps

Time-related bugs can be difficult to reproduce. Keep time handling explicit in applications that operate across regions or systems.

---

# 22. Resource management and cleanup

Production-quality programs need predictable cleanup.

Study:

- context managers
- `try` / `finally`
- file/resource ownership
- subprocess cleanup
- temporary resources
- application shutdown

Ask this question whenever you open or acquire something:

> **Who is responsible for releasing it, and what happens if an exception occurs?**

---

# 23. Exception design

The DevOps track introduces exception handling. Here, go deeper.

Learn:

- exception hierarchy
- catching the narrowest useful exception
- raising exceptions
- exception chaining
- custom exceptions
- preserving context
- cleanup with `finally`
- when to let an exception propagate

Example custom exception:

```python
class ConfigurationError(Exception):
    """Raised when application configuration is invalid."""
```

Avoid this pattern:

```python
try:
    do_work()
except Exception:
    pass
```

It can hide failures and make operational debugging much harder.

---

# 24. Logging architecture

Move beyond simply calling `logging.info()`.

Learn:

- loggers
- handlers
- formatters
- logging levels
- logger hierarchy
- configuration
- structured logging concepts
- avoiding duplicate handlers
- avoiding secrets and credentials in logs

A reusable library should generally avoid forcing an application's global logging configuration on its users.

---

# 25. Debugging and tracebacks

Learn to read a traceback from the exception type and message through the stack frames to the line that needs investigation.

Study standard-library debugging tools such as:

- `pdb`
- `traceback`
- `faulthandler`
- `inspect`
- `sys`

### Debugging workflow

```text
Reproduce
   ↓
Read the traceback
   ↓
Identify the failing boundary
   ↓
Inspect inputs/state
   ↓
Create a minimal reproduction
   ↓
Fix
   ↓
Add a regression test
```

Do not treat debugging as randomly adding print statements. Make the failure observable and reproducible.

---

# 26. Performance and profiling

Do not optimize based only on intuition.

Learn the difference between:

- algorithmic complexity
- I/O cost
- CPU cost
- memory use
- startup cost
- repeated work

Useful standard-library tools include:

- `timeit` for measuring small code snippets
- `cProfile` and `pstats` for profiling
- `tracemalloc` for tracing memory allocations

### Good workflow

```text
Measure
  ↓
Identify the bottleneck
  ↓
Change one thing
  ↓
Measure again
  ↓
Keep the change only if it helps
```

A faster implementation that is much harder to understand is not automatically a better production solution.

---

# 27. Advanced command-line applications

Extend your `argparse` knowledge into:

- subcommands
- reusable argument definitions
- validation
- meaningful exit codes
- help text
- stdin/stdout/stderr
- configuration precedence

A useful CLI should make success and failure understandable to both humans and automation systems.

---

# 28. Python project architecture

For larger programs, separate responsibilities instead of creating one giant script.

A practical design might separate:

```text
CLI layer
   ↓
configuration layer
   ↓
application/service layer
   ↓
integration layer
   ↓
external systems
```

For example, an infrastructure inventory tool might separate:

- argument parsing
- configuration loading
- API client
- resource transformation
- output formatting
- logging
- error handling

This makes individual components easier to test.

---

# 29. Advanced subprocess usage

The DevOps track introduces `subprocess`. Continue with:

- command arguments versus shell command strings
- standard input/output/error
- exit status
- timeouts
- environment handling
- text versus bytes
- process failures
- pipeline design

Treat external commands as unreliable boundaries. Check their results and handle failure deliberately.

Never construct shell commands from untrusted input without understanding the security consequences of shell interpretation.

---

# 30. Advanced file and filesystem programming

Continue with:

- `pathlib`
- `shutil`
- `tempfile`
- `glob`
- file metadata
- permissions and platform differences
- atomicity considerations
- encoding
- newline behavior

Practice building a file-processing utility that:

1. discovers files safely,
2. validates inputs,
3. processes them incrementally,
4. reports failures,
5. leaves the filesystem in a predictable state.

---

# 31. Unicode and text encoding

Learn the difference between:

```text
text
  ↓ encode
bytes
  ↓ decode
text
```

Understand:

- Unicode code points
- UTF-8
- encoding errors
- text versus binary files
- explicit encodings

Do not assume that every external system uses the same encoding.

---

# 32. Optional advanced language topics

These topics are valuable for advanced Python developers but are not prerequisites for most DevOps automation.

## Descriptors

Learn how attribute access can be customized through the descriptor protocol.

## Abstract base classes

Study `abc` when you need explicit interfaces or shared contracts across implementations.

## Metaclasses

Metaclasses control class creation and are powerful, but they are rarely necessary in ordinary automation code. Learn them to understand frameworks and advanced Python internals rather than because every project needs them.

## Structural pattern matching

Study `match` / `case` when it makes structured branching clearer.

These topics belong near the end of the learning path because they are easier to understand after the Python object model is familiar.

---

# 33. Complete Python project ideas

Build these in increasing difficulty.

### Project 1 — Reusable configuration library

Create a package that:

- loads configuration
- validates required values
- exposes typed models
- raises clear custom exceptions
- has unit tests

### Project 2 — Concurrent service checker

Build a tool that checks many endpoints and compares:

- sequential execution
- thread-based execution
- asynchronous execution

Measure the implementations instead of assuming one is faster.

### Project 3 — SQLite inventory application

Build a small CLI that stores infrastructure inventory in SQLite and supports querying and updates.

### Project 4 — Python package

Turn a useful automation library into an installable project with metadata, tests, documentation, and a CLI entry point.

### Project 5 — Production-style automation service

Combine:

- configuration
- typed models
- API integration
- structured logging
- error handling
- retries/timeouts where appropriate
- testing and mocks
- concurrency where justified
- packaging

Document the architecture and operational failure modes.

---

# 34. Exercises by skill level

## Intermediate

- write a custom iterator
- write a generator pipeline
- create a decorator using `functools.wraps`
- implement a context manager
- convert a data model to a dataclass
- add type annotations to an existing project

## Advanced

- design a package with clear module boundaries
- write unit tests with mocks for an external API
- build a concurrent batch processor
- build an `asyncio` version of an I/O-bound experiment
- profile a deliberately inefficient program
- add SQLite persistence to a CLI application

## Expert-oriented

- implement a small descriptor
- investigate import behavior in a multi-package project
- compare composition and inheritance for a real application model
- build a plugin-style architecture and document its trade-offs
- inspect a traceback and produce a minimal reproducible example

---

# 35. Common mistakes in advanced Python

- creating classes when a simple function would be clearer
- using inheritance where composition is simpler
- hiding important behavior behind decorators
- confusing iterables with iterators
- materializing huge collections unnecessarily
- using concurrency without measuring the workload
- mixing synchronous and asynchronous APIs without understanding the boundary
- mocking implementation details instead of external boundaries
- using type annotations as if they were automatic runtime validation
- creating circular imports
- putting substantial work in module import side effects
- catching broad exceptions and hiding failures
- optimizing before measuring
- using advanced language features merely to appear sophisticated

Advanced Python should make software easier to reason about, not harder.

---

# 36. What you should be able to do after this track

A learner who completes this continuation should be able to:

- design reusable Python modules and packages
- choose between functions, classes, and composition appropriately
- understand Python's iterator and object models
- use generators for streaming workflows
- write and reason about decorators and context managers
- model structured data with dataclasses and type annotations
- build testable code around external dependencies
- understand the major Python concurrency models
- write basic asynchronous applications with `asyncio`
- use SQLite from Python for appropriate local persistence
- debug tracebacks systematically
- profile code before optimizing
- package a reusable Python project
- reason about resource cleanup, errors, and operational failure modes

You do **not** need to master every advanced topic before returning to DevOps projects. The goal is to give you the depth to understand and maintain larger Python systems.

---

# 37. Official sources

This guide is synthesized for this repository from authoritative Python documentation. The sources are learning inputs; this README is the structured learner material.

- Python Tutorial: https://docs.python.org/3/tutorial/
- Python Standard Library: https://docs.python.org/3/library/
- Classes, iterators, and generators: https://docs.python.org/3/tutorial/classes.html
- Modules and packages: https://docs.python.org/3/tutorial/modules.html
- Errors and exceptions: https://docs.python.org/3/tutorial/errors.html
- `dataclasses`: https://docs.python.org/3/library/dataclasses.html
- `typing`: https://docs.python.org/3/library/typing.html
- `functools`: https://docs.python.org/3/library/functools.html
- `asyncio`: https://docs.python.org/3/library/asyncio.html
- `concurrent.futures`: https://docs.python.org/3/library/concurrent.futures.html
- `unittest.mock`: https://docs.python.org/3/library/unittest.mock.html
- `sqlite3`: https://docs.python.org/3/library/sqlite3.html
- Python Packaging User Guide: https://packaging.python.org/en/latest/

## Source boundary

The exact behavior of a Python feature depends on the Python version being used. When a version-specific feature, API, or behavior matters, consult the corresponding official documentation for that Python version.

Where this guide presents project structures, learning order, exercises, or design advice, those are **repository design decisions**, not official Python requirements.

---

# Completion checklist

### Functions and object model

- [ ] I understand scopes, namespaces, and object references.
- [ ] I can use advanced function parameters.
- [ ] I understand closures and decorators.

### OOP

- [ ] I can design classes and instances.
- [ ] I understand inheritance and composition.
- [ ] I can use special methods appropriately.

### Iteration

- [ ] I understand the iterator protocol.
- [ ] I can write generators.
- [ ] I understand lazy versus eager computation.

### Project structure

- [ ] I can organize modules and packages.
- [ ] I understand import behavior at a practical level.
- [ ] I can create a reusable Python package.

### Quality

- [ ] I can write tests with mocks when appropriate.
- [ ] I can design useful custom exceptions.
- [ ] I can debug tracebacks systematically.
- [ ] I can profile before optimizing.

### Concurrency

- [ ] I understand the difference between threads, processes, and asyncio.
- [ ] I can choose a concurrency model based on workload requirements.

### Data and applications

- [ ] I can model data with dataclasses and type annotations.
- [ ] I can use SQLite for an appropriate application.
- [ ] I understand resource cleanup and failure handling.

---

**Next step:** return to the wider DevOps roadmap and apply these deeper Python skills where they provide real value. The next sections of the repository should build on this foundation rather than repeat it.
