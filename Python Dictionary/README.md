# Python Dictionary

A learner-first Python curriculum built around a simple rule:

> **Learn the Python needed for DevOps first, then continue into Complete Python.**

This file is the roadmap and study guide for both branches. It is intentionally more than a list of links: each section explains what to learn, why it matters, what to practice, and where it fits in the DevOps journey.

## Learning method

Use this loop for every topic:

**Understand → Write → Run → Break → Fix → Practice → Apply**

Do not move forward because you can recognize syntax. Move forward when you can explain the idea, write a small example without copying it, diagnose a common failure, and use the concept in an automation task.

## Two-stage structure

| Path | What it contains |
|---|---|
| **Python for DevOps** | The Python foundation and standard-library skills that are directly useful for scripting, automation, configuration, APIs, processes, files, logging, testing, and operational tooling. |
| **Complete Python** | The broader language and standard-library topics that continue from the DevOps foundation. |

The second path is a continuation, not a second beginner course. Shared fundamentals should be learned once.

---

# Part I — Python for DevOps

## 1. Python fundamentals

### Learn

- What Python is and how a Python program executes at a beginner level
- Interactive interpreter versus `.py` programs
- Comments and readable code
- Variables and assignment
- Names and objects at a practical level
- Integers and floating-point numbers
- Booleans
- Strings
- `None`
- Arithmetic, comparison, Boolean, and assignment operators
- Type conversion and checking
- `input()` and `print()`
- f-strings

### You should be able to

- accept input and turn it into the required type;
- store configuration-like values;
- calculate and compare values;
- produce useful status output;
- explain basic Python errors instead of guessing at fixes.

### Practice

1. Read a service name and print a status message.
2. Convert a numeric string to an integer and calculate a result.
3. Format a deployment summary with an f-string.
4. Classify a numeric value as low, normal, or high.

---

## 2. Conditions and control flow

### Learn

- `if`, `elif`, `else`
- comparison operators
- Boolean expressions and truth-value testing
- nested conditions
- `for` loops
- `while` loops
- `range()`
- `break`, `continue`, `pass`
- loop `else` clauses
- `match` statements as a later language feature

### DevOps use

Automation repeatedly performs the same operation over hosts, files, services, records, or API results and makes decisions about each item.

### Example

```python
servers = ["web-01", "web-02", "db-01"]

for server in servers:
    if server.startswith("web-"):
        print(f"Checking web server: {server}")
    else:
        print(f"Checking other server: {server}")
```

The reusable pattern is **iterate → inspect → decide → act**.

### Practice

- scan service names and report matches;
- count failed statuses;
- stop processing when a required condition is reached;
- read lines and count selected events.

---

## 3. Functions

### Learn

- defining and calling functions
- parameters and return values
- local scope
- default arguments
- keyword arguments
- positional arguments
- positional-only and keyword-only parameters
- `*args` and `**kwargs`
- argument unpacking
- lambda expressions
- docstrings
- function annotations

### DevOps use

Functions let a script separate configuration loading, validation, API calls, reporting, and other operations into reusable units.

### Practice

Write functions for:

- configuration validation;
- host filtering;
- report formatting;
- API-result validation;
- deployment summaries.

---

## 4. Collections and data handling

### Learn

- lists
- tuples
- sets
- dictionaries
- indexing and slicing
- membership testing
- mutability versus immutability
- list methods such as `append`, `extend`, `insert`, `remove`, `pop`, `clear`, `count`, `index`, `sort`, `reverse`, and `copy`
- dictionary keys and values
- set operations
- iteration
- list comprehensions
- nested collections
- unpacking

### Choosing a collection

| Need | Common choice |
|---|---|
| Ordered collection that changes | `list` |
| Fixed sequence | `tuple` |
| Unique values and membership operations | `set` |
| Key/value lookup | `dict` |

This is a learning guide, not a rule that overrides the requirements of a real program.

### Example

```python
servers = {
    "web-01": "healthy",
    "web-02": "degraded",
    "db-01": "healthy",
}

for name, status in servers.items():
    print(f"{name}: {status}")
```

### Practice

- represent service state with a dictionary;
- find unhealthy services;
- remove duplicate hostnames with a set;
- transform filenames with a comprehension.

---

## 5. Strings and text processing

### Learn

- indexing and slicing
- string methods
- searching and replacing
- `split()` and `join()` concepts
- whitespace handling
- f-string formatting
- text normalization
- regular expressions later, after ordinary string operations are understood

### DevOps use

Logs, command output, identifiers, configuration files, and API data frequently contain text. Learn to transform text without assuming more structure than the input actually guarantees.

### Practice

- count error messages in a log;
- extract fields from a predictable line format;
- normalize whitespace before comparison;
- build a readable report from structured values.

---

## 6. Files and filesystem automation

### Learn

- opening and closing files
- reading and writing text
- file modes
- `with` statements
- context managers at a practical level
- `pathlib.Path`
- directory traversal
- existence checks
- file and directory operations
- temporary files/directories
- file discovery with glob-style patterns
- JSON serialization and deserialization

Python's official documentation describes `pathlib` as an object-oriented filesystem-path interface; `Path` is the usual starting point for concrete path work.

### Example

```python
from pathlib import Path

config = Path("config.json")

if config.exists():
    print(f"Found configuration: {config}")
```

### Practice project — configuration updater

Build a program that locates a configuration file, reads it safely, validates required values, changes one selected setting, writes the result, and reports what happened.

---

## 7. Environment variables and process environment

### Learn

- what environment variables are;
- reading them with the standard library;
- defaults and required values;
- configuration versus secrets;
- avoiding hard-coded credentials;
- passing environment data to child processes.

### Practice

Write a script that reads a non-secret configuration value from the environment and clearly reports whether a required value is present.

**Security rule:** never put real credentials, tokens, private keys, or production secrets into examples or Git history.

---

## 8. Command-line automation

### Learn

- `sys.argv` at a basic level
- `argparse`
- positional arguments
- options and flags
- boolean flags
- help and usage output
- argument validation
- subcommands as an advanced CLI pattern

Python's `argparse` module is the standard-library facility for basic command-line interfaces. It parses arguments from `sys.argv`, generates help/usage messages, and reports invalid arguments.

### Example

```python
import argparse

parser = argparse.ArgumentParser(description="Check a service")
parser.add_argument("service")
parser.add_argument("--verbose", action="store_true")
args = parser.parse_args()

print(f"Checking {args.service}")
```

### Practice project

Create a CLI checker that accepts a target, validates the input, performs a check, and reports a useful result to both a human and an automated caller.

---

## 9. Running operating-system commands

### Learn

- process execution versus ordinary function calls
- `subprocess`
- `subprocess.run()`
- stdout and stderr
- return codes
- timeouts
- environment passing
- command failure handling
- safe argument construction

Python documents `subprocess.run()` as the recommended approach for cases it can handle. It waits for completion and returns a `CompletedProcess`; `check=True` raises `CalledProcessError` for a non-zero exit status.

### Example

```python
import subprocess

result = subprocess.run(
    ["python", "--version"],
    capture_output=True,
    text=True,
)

print(result.stdout)
print(result.returncode)
```

### Practice

Build a wrapper that runs a documented command, captures its result, handles an expected failure, and reports the exit status.

---

## 10. Errors and exceptions

### Learn

- syntax errors versus exceptions
- reading tracebacks
- exception types
- `try`, `except`, `else`, `finally`
- raising exceptions
- custom exceptions
- exception chaining
- cleanup

Python's tutorial distinguishes syntax errors from exceptions raised during execution. A traceback provides context about where the failure occurred and the exception type identifies the kind of failure.

### DevOps rule

Do not hide failures merely to make a script look successful. Handle an exception when the program can recover or produce a useful alternative; otherwise keep the failure visible and actionable.

### Practice

Create a program that validates input, handles an expected failure, reports the cause clearly, performs cleanup, and communicates failure to its caller.

---

## 11. Modules and packages

### Learn

- importing modules
- creating reusable modules
- `__name__`
- running a module as a script
- package structure
- imports between packages
- separating reusable logic from CLI entry points
- avoiding circular dependencies

### Practice

Split an automation tool into modules for configuration, reusable operations, reporting, and its command-line entry point.

---

## 12. Virtual environments and packages

### Learn

- dependency isolation
- virtual environments
- `.venv`
- package installation concepts
- reproducible dependencies
- why virtual environments should not be committed to Git

Python's `venv` module creates lightweight isolated environments. The official documentation describes them as disposable and recommends recreating them rather than moving/copying them; they should not be checked into source control.

### Example

```text
python -m venv .venv
```

The exact activation command depends on the operating system and shell; use the current Python documentation for platform-specific activation instructions.

### Practice

Create a small project in an isolated environment and document how another learner can recreate it.

---

## 13. JSON and configuration data

### Learn

- JSON objects and arrays as represented in Python
- loading JSON
- writing JSON
- serialization/deserialization
- validating configuration shape
- separating configuration from code

### Practice project

Build a configuration manager that loads JSON, checks required fields, reports invalid configuration clearly, and keeps configuration handling separate from business logic.

---

## 14. YAML and configuration formats

### Learn

- why configuration formats exist
- YAML concepts required for DevOps
- safe parsing principles
- configuration versus executable code
- when structured JSON is sufficient

YAML library behavior should be taught from the official documentation of the chosen library when this section becomes an implementation file. Do not invent a YAML API.

---

## 15. HTTP, REST APIs, and automation

### Learn

- request/response model
- HTTP methods and status codes at a practical level
- headers
- query parameters
- request bodies
- JSON responses
- authentication concepts
- timeouts
- expected request failures
- retry design as an advanced topic

### DevOps use

DevOps platforms commonly expose APIs. The goal is to turn repeatable operational workflows into automation.

### Practice project

Build an API-monitoring script that calls a documented endpoint, checks the response, parses returned data, reports success/failure, and handles expected request failures. The exact endpoint and client library must be selected from authoritative documentation when implemented.

---

## 16. Logging

### Learn

- why logging is different from `print()`
- loggers
- log levels
- handlers
- formatters
- module-level loggers
- logging configuration
- operational context
- avoiding secrets in logs

Python's standard `logging` module provides a flexible event logging system. Its documentation shows the common pattern of using `logging.getLogger(__name__)` in modules and configuring the logging system at the application level.

### Example

```python
import logging

logger = logging.getLogger(__name__)
logger.info("Starting configuration validation")
```

### Practice

Convert a small automation script from print-only status messages to useful logging.

---

## 17. Testing automation

### Learn

- why automation needs tests
- assertions
- unit tests
- test cases
- fixtures and cleanup
- suites
- runners
- testing failure paths
- testing filesystem logic
- mocking concepts

Python's `unittest` framework supports test automation, setup/cleanup, test cases, suites, and runners.

### Practice project

Test a configuration validator with valid configuration, missing fields, invalid values, boundary cases, and expected exceptions.

---

## 18. Practical DevOps projects

### Project 1 — Log analyzer

Read a log, count selected events, identify errors, summarize the result, and test the parsing logic.

### Project 2 — Configuration validator

Load structured configuration, validate required fields and values, report actionable errors, and test failure cases.

### Project 3 — CLI automation tool

Use `argparse`, validate input, perform an operation, log what happened, and handle expected failures.

### Project 4 — API automation

Call a documented API, parse JSON, handle request failures, log important events, and test response handling.

### Project 5 — Infrastructure automation

Apply the same Python foundations to an actual cloud/platform tool. The client, authentication, API, resource model, and commands must be learned from that platform's official documentation.

### Project 6 — Kubernetes automation

The official `kubernetes-client/python` project is an applied reference for Python-based Kubernetes API automation. It belongs after the Python foundation because the client is an application of Python, not a replacement for learning Python.

---

# Part II — Complete Python

**Complete Python continues from Python for DevOps. It must not repeat the same beginner material.**

## 19. Deeper object model

Learn:

- names and objects
- identity versus equality
- mutability
- references
- scope and namespaces
- attribute access
- object lifetime at a conceptual level

These concepts become valuable when debugging unexpected state changes in larger programs.

## 20. Classes and object-oriented programming

Learn:

- classes and instances
- attributes and methods
- class and instance variables
- constructors
- inheritance
- multiple inheritance
- overriding
- composition
- properties
- special methods
- encapsulation conventions

Use OOP when it improves the design, not merely because classes exist.

## 21. Iterators and generators

Learn:

- iterable versus iterator
- `iter()` and `next()` concepts
- generator functions
- `yield`
- generator expressions
- lazy processing

This is useful when processing large logs, inventories, files, or result sets incrementally.

## 22. Decorators and closures

Learn:

- functions as objects
- higher-order functions
- closures
- decorator syntax
- wrapping behavior
- preserving function metadata

Understand the function model before using decorators in production code.

## 23. Context managers

Learn:

- resource lifecycle
- `with`
- context manager protocol
- `__enter__` and `__exit__`
- `contextlib`
- custom context managers

This extends the cleanup concepts learned through file handling.

## 24. Type hints and static analysis

Learn:

- basic annotations
- collection types
- optional values
- unions
- type aliases
- generic concepts
- protocols later
- type checking as a development practice

Type hints improve readability and tooling; they do not replace runtime validation.

## 25. Python data structures and algorithmic foundations

Learn Python implementations and trade-offs for:

- stacks
- queues and deques
- heaps
- hash-based lookup
- trees
- graphs

The full algorithm curriculum belongs in **DSA Dictionary** so that DSA explanations and implementations have one canonical home.

## 26. Functional-style tools

Learn:

- `map`
- `filter`
- `zip`
- `enumerate`
- `sorted`
- `any` / `all`
- comprehensions
- `itertools`
- `functools`
- `operator`

The objective is clear data transformation, not clever one-line code.

## 27. Regular expressions

Learn:

- patterns
- character classes
- groups
- quantifiers
- anchors
- searching and matching
- substitution
- common mistakes

Use a real parser when the input has a defined structured format; regular expressions are not a universal parser.

## 28. Dates and times

Learn:

- date/time objects
- parsing and formatting
- durations
- comparisons
- time zones
- timestamp handling

These concepts matter for logs, monitoring, scheduling, and incident analysis.

## 29. Concurrency and parallel execution

Learn progressively:

- processes versus threads
- `threading`
- `multiprocessing`
- `concurrent.futures`
- queues
- synchronization concepts
- asynchronous programming later

Measure the bottleneck before adding concurrency.

## 30. Networking and sockets

Learn:

- client/server concepts
- sockets
- connection lifecycle
- timeouts
- basic network troubleshooting concepts

This goes deeper than ordinary API consumption and is useful for platform/system-oriented work.

## 31. Databases and persistence

Learn:

- persistence concepts
- relational database fundamentals
- SQL from Python
- transactions
- connection lifecycle
- parameterized queries
- basic data modeling

The exact database driver should be selected from its official documentation when implementation begins.

## 32. Packaging and project structure

Learn:

- package structure
- imports
- project metadata
- dependency management
- build/distribution concepts
- reproducible environments
- publishing concepts

Use current Python packaging guidance rather than copying an outdated layout.

## 33. Debugging and profiling

Learn:

- deep traceback reading
- debugging techniques
- logging for diagnosis
- assertions
- profiling concepts
- measurement before optimization
- CPU, memory, I/O, and network bottlenecks

Optimize because you have evidence, not because a technique sounds advanced.

## 34. Security-aware Python

Learn:

- secret handling
- input validation
- safe subprocess usage
- safe file handling
- dependency awareness
- unsafe deserialization risks
- least-privilege thinking
- avoiding sensitive data in logs

Security-specific behavior must be verified against authoritative documentation for the library or platform involved.

---

# Learning checkpoints

## Ready to leave Python for DevOps?

You should be able to:

- write a small script without blindly copying a template;
- use variables, conditions, loops, functions, lists, dictionaries, and sets;
- read and write files safely;
- work with paths using `pathlib`;
- handle expected exceptions;
- build a CLI with `argparse`;
- execute a process with `subprocess` and inspect its result;
- use environment variables for configuration;
- parse JSON;
- structure code into modules;
- create a virtual environment;
- write basic automated tests;
- use logging for operational visibility;
- explain both success and failure behavior of your automation.

## Ready to call Complete Python complete?

You should additionally be comfortable with classes, object behavior, iterators/generators, decorators, context managers, type hints, advanced standard-library tools, concurrency concepts, networking, packaging, debugging/profiling, and security-aware design.

---

# What DevOps learners can defer

A DevOps engineer does not need to master every corner of Python before becoming productive. Advanced OOP patterns, metaprogramming, deep interpreter internals, advanced descriptor behavior, complex packaging internals, advanced concurrency, advanced networking, and specialized standard-library modules can be deferred until the work requires them.

**Defer does not mean ignore.** It means the learning order prioritizes practical value first.

---

# Source-derived design

The official Python tutorial provides the language progression: introduction, control flow, functions, data structures, modules, input/output, errors/exceptions, classes, standard-library tours, and virtual environments/packages.

The `iam-veeramalla/python-for-devops` source adds a practical DevOps sequence around lists, loops, dictionaries/sets, file operations, AWS automation, remote task automation, REST APIs, JSON/YAML configuration, scheduled automation, and project-style exercises.

This repository synthesizes those materials into a **DevOps-first → Complete Python** progression. That ordering is a repository design decision, not an official definition of what every DevOps engineer must know.

## References used while building this file

- Python Tutorial: https://docs.python.org/3/tutorial/
- Python Standard Library: https://docs.python.org/3/library/
- `pathlib`: https://docs.python.org/3/library/pathlib.html
- `subprocess`: https://docs.python.org/3/library/subprocess.html
- `argparse`: https://docs.python.org/3/library/argparse.html
- `logging`: https://docs.python.org/3/library/logging.html
- `unittest`: https://docs.python.org/3/library/unittest.html
- `venv`: https://docs.python.org/3/library/venv.html
- Python for DevOps source: https://github.com/iam-veeramalla/python-for-devops
- Kubernetes Python client: https://github.com/kubernetes-client/python
