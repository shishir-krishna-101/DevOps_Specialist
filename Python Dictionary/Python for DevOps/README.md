# Python for DevOps

A practical, learner-first Python curriculum for DevOps engineers.

This guide focuses on the Python skills that are most useful for automation, infrastructure work, operational tooling, APIs, configuration, testing, and troubleshooting. It is designed so that a learner can study the material directly from this repository instead of treating it as a list of links.

The **Complete Python** path is the continuation. It covers Python topics that are useful for deeper software development but are not required before a DevOps engineer can become productive with Python.

---

## How to use this guide

For every topic, follow this cycle:

1. **Understand** the concept.
2. **Type and run** the example yourself.
3. **Change** the example and observe what happens.
4. **Solve** the exercises without looking at the solution first.
5. **Apply** the idea to a DevOps problem.
6. **Test** the result.
7. **Review** the mistakes and improve the script.

Do not try to memorize Python syntax. Learn the problem that a construct solves and practice using it.

---

# 1. Python mental model

Python is a general-purpose programming language. For DevOps work, it is especially useful as a glue language: a script can read configuration, inspect files, call an API, execute another process, transform data, and report the result.

A useful mental model is:

```text
Input
  ↓
Parse / validate
  ↓
Transform
  ↓
Perform an action
  ↓
Check the result
  ↓
Log / report
```

A good automation script should make each stage understandable.

### Example

```python
from pathlib import Path

config = Path("config.txt")

if not config.exists():
    raise FileNotFoundError(config)

for line in config.read_text().splitlines():
    print(line)
```

The important ideas are not the number of lines. The script identifies an input, checks it, reads it, and processes it.

---

# 2. Running Python

You should understand three common ways to work with Python:

- interactive interpreter
- Python script files
- modules and packages

A Python script normally has a `.py` extension.

Example:

```python
print("Hello, DevOps")
```

Run it with the Python interpreter available in your environment.

For reusable programs, prefer a script with a clear entry point:

```python
def main():
    print("Starting automation")


if __name__ == "__main__":
    main()
```

The `if __name__ == "__main__":` pattern lets a module provide reusable functions while also being executable as a script.

---

# 3. Variables and basic data types

A variable name refers to an object.

```python
server_count = 3
service_name = "nginx"
is_healthy = True
```

Common built-in types you should know early:

| Type | Example | Typical DevOps use |
|---|---|---|
| `int` | `3` | counts, ports, retry limits |
| `float` | `2.5` | measurements and ratios |
| `str` | `"prod"` | names, paths, API data |
| `bool` | `True` | feature/state checks |
| `None` | `None` | absence of a value |
| `list` | `["web", "db"]` | ordered collections |
| `tuple` | `("prod", "web")` | fixed sequences |
| `set` | `{"prod", "dev"}` | unique values |
| `dict` | `{"env": "prod"}` | structured key/value data |

Check a value with `type()` while learning:

```python
value = "production"
print(type(value))
```

### Mutable vs immutable

Lists, dictionaries, and sets are mutable. Their contents can be changed after creation.

Strings and tuples are immutable.

```python
servers = ["web-1", "web-2"]
servers.append("web-3")

name = "web"
# A new string is produced when you create a changed value.
```

Understanding mutability becomes important when functions receive collections.

---

# 4. Operators and expressions

Know arithmetic, comparison, logical, membership, and identity operators.

```python
retries = 2
limit = 3

print(retries + 1)
print(retries < limit)
print(retries < limit and limit > 0)
```

### `==` versus `is`

`==` compares values. `is` checks object identity.

Use `==` when you want to compare values. `is` is commonly used when checking for `None`:

```python
result = None

if result is None:
    print("No result")
```

Do not use `is` as a general replacement for `==`.

---

# 5. Strings

Strings are central to automation because paths, command output, configuration, logs, API fields, and identifiers are often text.

```python
environment = "production"
service = "api"

message = f"Deploying {service} to {environment}"
print(message)
```

Learn:

- indexing
- slicing
- `split()`
- `join()`
- `strip()`
- `replace()`
- `lower()` / `upper()`
- `startswith()` / `endswith()`
- membership with `in`
- f-strings

### Example: parse simple log lines

```python
line = "ERROR database connection failed"
parts = line.split()

if parts and parts[0] == "ERROR":
    print("An error was found")
```

Do not assume that every real log format can be safely parsed with `split()`. Use the format actually produced by the system.

---

# 6. Conditions

Use `if`, `elif`, and `else` to make decisions.

```python
status = 503

if status >= 500:
    print("Server error")
elif status >= 400:
    print("Client error")
else:
    print("Request did not return a 4xx/5xx status")
```

Good automation code makes the decision rules obvious.

Avoid deeply nested conditions when a simpler structure communicates the intent better.

---

# 7. Loops

Python provides `for` and `while` loops.

### `for`

Use `for` when iterating over a collection or iterable.

```python
services = ["api", "worker", "scheduler"]

for service in services:
    print(service)
```

### `range()`

```python
for attempt in range(3):
    print(attempt)
```

### `while`

Use `while` when repetition depends on a condition.

```python
attempt = 0

while attempt < 3:
    print(f"Attempt {attempt}")
    attempt += 1
```

Always make sure a `while` loop can eventually terminate.

### `break` and `continue`

```python
for service in services:
    if service == "worker":
        continue
    print(service)
```

Use these deliberately; excessive control-flow jumps can make automation harder to reason about.

---

# 8. Lists

Lists maintain an ordered collection and are one of the most frequently used Python structures.

```python
servers = ["web-1", "web-2"]
servers.append("web-3")
```

Important operations include:

- `append()` — add one item
- `extend()` — add items from an iterable
- `insert()` — insert at an index
- `remove()` — remove the first matching value
- `pop()` — remove and return an item
- `sort()` — sort in place
- `reverse()` — reverse in place
- `count()` — count occurrences
- `index()` — find a value
- `copy()` — make a shallow copy

### `append()` versus `extend()`

```python
items = [1, 2]
items.append([3, 4])
print(items)  # [1, 2, [3, 4]]
```

Whereas:

```python
items = [1, 2]
items.extend([3, 4])
print(items)  # [1, 2, 3, 4]
```

### List comprehensions

```python
healthy = [s for s in services if s != "failed"]
```

Use comprehensions when the transformation is easy to understand. Prefer a normal loop when the logic becomes complicated.

---

# 9. Dictionaries

Dictionaries store key/value pairs and are extremely important for configuration and API data.

```python
server = {
    "name": "web-1",
    "environment": "production",
    "healthy": True,
}

print(server["name"])
```

Use `.get()` when a key may be absent:

```python
port = server.get("port", 80)
```

Useful methods include:

- `get()`
- `keys()`
- `values()`
- `items()`
- `update()`
- `pop()`

### Iterating over configuration

```python
for key, value in server.items():
    print(key, value)
```

A dictionary is often the first useful representation when converting JSON into Python data.

---

# 10. Sets and tuples

Sets contain unique elements.

```python
environments = {"dev", "test", "prod"}
```

They are useful when you care about membership or uniqueness rather than ordering.

Tuples are immutable sequences:

```python
server = ("web-1", "production")
```

They are useful when a fixed grouping of values is appropriate.

---

# 11. Functions

Functions let you turn repeated logic into reusable units.

```python
def is_healthy(status_code):
    return 200 <= status_code < 300
```

Call it with:

```python
print(is_healthy(200))
```

Learn:

- parameters
- return values
- default arguments
- keyword arguments
- positional arguments
- variable-length arguments
- scope
- docstrings
- annotations

### Default arguments

```python
def retry_count(limit=3):
    return limit
```

### Keyword arguments

```python
def deploy(service, environment):
    print(service, environment)


deploy(service="api", environment="production")
```

### `*args` and `**kwargs`

Understand these before using them heavily. They allow functions to accept variable numbers of positional or keyword arguments.

### Good DevOps function design

Prefer functions that do one understandable job:

```text
read_config()
validate_config()
build_command()
run_command()
report_result()
```

This makes scripts easier to test and troubleshoot.

---

# 12. Modules and imports

Put related functions into modules instead of keeping every script in one huge file.

Example structure:

```text
automation/
├── main.py
├── config.py
└── checks.py
```

Import a module:

```python
import checks

checks.run_health_check()
```

Or import a specific name:

```python
from checks import run_health_check
```

Prefer clear imports. Avoid wildcard imports such as `from module import *` in production automation.

---

# 13. Exceptions and error handling

Python distinguishes syntax errors from exceptions raised while executing code.

For automation, error handling is critical because a script that silently continues after a failed operation can be worse than one that stops.

```python
try:
    value = int("not-a-number")
except ValueError:
    print("Invalid number")
```

You should understand:

- `try`
- `except`
- `else`
- `finally`
- `raise`
- exception classes
- exception propagation
- custom exceptions

### Catch specific exceptions

Prefer:

```python
try:
    value = int(text)
except ValueError:
    print("Expected a number")
```

rather than catching every possible exception without a reason.

### Raise an error when a precondition fails

```python
def require_environment(name):
    if not name:
        raise ValueError("Environment name is required")
```

The Python documentation explains that exceptions have types and tracebacks that identify where an error occurred. Learn to read the traceback instead of treating it as noise.

---

# 14. Files

DevOps automation frequently reads and writes configuration, logs, inventories, reports, and generated files.

A context manager is the preferred basic pattern for file access:

```python
with open("servers.txt", "r") as file:
    content = file.read()
```

The `with` statement ensures the file is cleaned up when the block exits.

For line-oriented processing:

```python
with open("servers.txt") as file:
    for line in file:
        server = line.strip()
        if server:
            print(server)
```

### Writing

```python
with open("report.txt", "w") as file:
    file.write("Deployment complete\n")
```

Choose the mode deliberately. Do not accidentally overwrite important data.

---

# 15. `pathlib`

`pathlib` provides object-oriented filesystem paths and is usually the clearest choice for modern Python filesystem code.

```python
from pathlib import Path

root = Path("logs")

for path in root.glob("*.log"):
    print(path)
```

Useful operations include:

```python
path.exists()
path.is_file()
path.is_dir()
path.read_text()
path.write_text("hello")
path.mkdir()
```

Build paths using path operations rather than manually concatenating platform-specific separators.

---

# 16. `os` and environment variables

The `os` module provides operating-system interfaces.

Environment variables are commonly used to provide configuration without hard-coding values into source code.

```python
import os

environment = os.environ.get("ENVIRONMENT", "development")
print(environment)
```

Never commit credentials or secrets simply because a script needs them.

Treat these as different concepts:

```text
configuration → ordinary application/automation settings
secret        → sensitive credential or key material
```

Do not print secrets to logs.

---

# 17. JSON

JSON is common in APIs and configuration systems.

```python
import json

payload = '{"service": "api", "healthy": true}'
data = json.loads(payload)

print(data["service"])
```

Convert Python data to JSON:

```python
output = json.dumps({"service": "api"})
```

For files:

```python
from pathlib import Path
import json

config = json.loads(Path("config.json").read_text())
```

Understand the distinction:

- `json.loads()` — JSON string → Python object
- `json.dumps()` — Python object → JSON string
- `json.load()` — file-like object → Python object
- `json.dump()` — Python object → file-like object

---

# 18. YAML and configuration

YAML is widely encountered in infrastructure and application configuration. Python does not provide a YAML parser in its standard library, so a YAML workflow requires an appropriate external package.

Before using a third-party parser:

1. identify the package actually used by the project;
2. read its official documentation;
3. understand safe parsing requirements;
4. pin/manage dependencies appropriately;
5. test malformed input.

Do not treat arbitrary YAML as trusted input merely because it is a configuration file.

A useful configuration design separates:

```text
configuration structure
        ↓
validation
        ↓
normalization
        ↓
application logic
```

---

# 19. Command-line interfaces

DevOps scripts are often called from terminals, CI jobs, schedulers, or other automation.

The standard library's `argparse` module provides a way to define command-line arguments and automatically generate help and usage messages.

Example:

```python
import argparse

parser = argparse.ArgumentParser(description="Check a service")
parser.add_argument("service")
parser.add_argument("--verbose", action="store_true")
args = parser.parse_args()

print(args.service)
print(args.verbose)
```

A good CLI should make these things clear:

- required inputs
- optional inputs
- defaults
- valid choices
- help text
- errors
- exit behavior

Do not bury configuration inside the source code when the value naturally belongs to the invocation or environment.

---

# 20. Calling external commands with `subprocess`

DevOps automation often needs to integrate with existing command-line tools.

Python's `subprocess` module can create processes, connect to their standard input/output/error streams, and obtain return codes.

For straightforward cases, the documentation recommends `subprocess.run()`.

Example:

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

### Important principle

Prefer passing command arguments as a sequence rather than constructing a shell command string unnecessarily.

Be especially careful with untrusted input and shell execution. Never assume that inserting user-controlled text into a command is safe.

### `check=True`

When appropriate:

```python
subprocess.run(command, check=True)
```

This causes a non-zero exit status to raise `CalledProcessError`.

Understand the difference between:

```text
process started successfully

and

command completed successfully
```

A process can start and still return a failure status.

---

# 21. Logging

Use logging rather than scattering `print()` statements through serious automation.

Python's standard `logging` module provides loggers, handlers, filters, and formatters.

Basic pattern:

```python
import logging

logger = logging.getLogger(__name__)

logger.info("Starting deployment")
logger.warning("Configuration is incomplete")
logger.error("Deployment failed")
```

A common application structure configures logging near the program entry point and creates module-level loggers with `getLogger(__name__)`.

Useful levels to understand:

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

Do not log passwords, tokens, private keys, or other sensitive values.

---

# 22. HTTP and REST APIs

DevOps engineers frequently automate systems through APIs.

Understand these concepts before writing API automation:

- URL
- HTTP method
- headers
- request body
- response body
- status code
- authentication
- authorization
- timeout
- retry behavior
- pagination
- rate limits
- JSON serialization

A conceptual request looks like:

```text
Python program
    ↓
HTTP request
    ↓
API endpoint
    ↓
HTTP response
    ↓
status + headers + body
    ↓
validate and process
```

Do not assume every API uses the same authentication mechanism or response format. Read the API's authoritative documentation.

### API automation checklist

Before automating an API, determine:

1. What endpoint is required?
2. Which HTTP method is required?
3. What authentication mechanism is required?
4. What request format is expected?
5. What response indicates success?
6. What errors can occur?
7. Are requests idempotent?
8. Is pagination required?
9. Are rate limits documented?
10. What should the script log?

---

# 23. Authentication and secrets

Authentication answers “who are you?” Authorization answers “what are you allowed to do?”

Python automation should not hard-code credentials into source files.

Prefer the secret/configuration mechanism supported by the environment and the service being automated.

Basic principles:

- do not commit secrets;
- do not print secrets;
- minimize permissions;
- avoid putting secrets in command-line arguments when the environment makes them visible to other processes/users;
- rotate credentials according to your organization's policy;
- use short-lived credentials where the platform supports them;
- fail clearly when required credentials are missing.

These are security practices, not substitutes for the authentication documentation of the service being automated.

---

# 24. Testing

Automation needs tests because a script that changes infrastructure should be treated as production software.

Python includes `unittest` in the standard library.

A test checks a specific expected behavior.

Example:

```python
import unittest


def is_healthy(status_code):
    return 200 <= status_code < 300


class HealthTest(unittest.TestCase):
    def test_success_status(self):
        self.assertTrue(is_healthy(200))

    def test_error_status(self):
        self.assertFalse(is_healthy(500))


if __name__ == "__main__":
    unittest.main()
```

Learn the ideas of:

- test case
- test fixture
- test suite
- test runner
- assertions
- setup and cleanup
- mocking external dependencies

### What to test in DevOps scripts

Test pure logic heavily:

```text
input → transformation → expected result
```

For external systems, isolate or mock where appropriate and separately test the integration behavior.

---

# 25. Iterators and generators — practical introduction

An iterable can provide values one at a time. Generators are useful when processing potentially large streams without creating an entire result collection at once.

```python
def read_servers(lines):
    for line in lines:
        server = line.strip()
        if server:
            yield server
```

Use a generator when the consumer can process values incrementally.

This is particularly useful for:

- large files
- streams of records
- log processing
- pipelines of transformations

The deeper iterator/generator model belongs in **Complete Python**, but a DevOps engineer should understand the basic pattern.

---

# 26. Useful standard-library modules for DevOps

The Python standard library contains many modules relevant to automation.

Prioritize these:

| Module | Why learn it |
|---|---|
| `pathlib` | filesystem paths |
| `os` | operating-system interfaces and environment |
| `shutil` | high-level filesystem operations |
| `subprocess` | external processes |
| `argparse` | command-line interfaces |
| `json` | JSON data |
| `logging` | application logging |
| `unittest` | testing |
| `re` | regular expressions |
| `datetime` | dates and times |
| `time` | time-related operations |
| `tempfile` | temporary files/directories |
| `glob` | pathname pattern matching |
| `csv` | CSV data |
| `hashlib` | hashes |
| `secrets` | security-oriented random values |
| `typing` | type annotations |
| `collections` | specialized collection types |
| `itertools` | iterator building blocks |
| `functools` | higher-order functions |

Do not try to memorize the whole standard library. Learn to recognize when a standard-library module is likely to solve a problem before writing your own utility.

---

# 27. Regular expressions

Regular expressions are useful for structured text matching, but they should not automatically be the first solution for every parsing problem.

Example:

```python
import re

match = re.search(r"ERROR", "2025-01-01 ERROR database unavailable")

if match:
    print("Error found")
```

Learn:

- literal matching
- character classes
- groups
- quantifiers
- anchors
- `search()`
- `match()`
- `findall()`
- `sub()`

For complex structured data, prefer a parser for the actual format instead of building a fragile regular expression.

---

# 28. Dates and times

Automation often needs timestamps, durations, and scheduling calculations.

Learn the `datetime` module and distinguish:

- date
- time
- datetime
- duration (`timedelta`)
- timezone-aware values
- timezone-naive values

Do not silently assume that a timestamp is in local time or UTC. Know what the source system provides.

---

# 29. CSV and tabular data

CSV is common in inventories, reports, exports, and operational data.

Use Python's `csv` module when the input is actually CSV rather than manually splitting every line on commas.

Why? CSV fields can contain commas, quotes, and escaped content that makes naïve `split(",")` parsing unreliable.

---

# 30. Configuration validation

A reliable automation program should validate configuration before performing destructive or expensive operations.

Think in stages:

```text
read configuration
        ↓
check required fields
        ↓
check types
        ↓
check allowed values
        ↓
normalize values
        ↓
perform operation
```

Example:

```python
def validate_config(config):
    required = ["environment", "service"]

    for key in required:
        if not config.get(key):
            raise ValueError(f"Missing configuration: {key}")
```

For real systems, validation rules should come from the system's documented configuration contract.

---

# 31. Idempotency

Idempotency is a critical automation concept.

A useful automation operation should be safe to run repeatedly when the intended state has already been reached.

Think about the difference between:

```text
create a resource
```

and:

```text
ensure the resource exists in the desired state
```

Whether an operation is actually idempotent depends on the system being automated. Do not claim that an API or command is idempotent without verifying its documentation.

When writing your own automation, explicitly ask:

1. What is the desired state?
2. How do I detect the current state?
3. What happens if I run this twice?
4. Can a partial failure leave the system in an unexpected state?
5. Can the operation safely be retried?

---

# 32. Retries and timeouts

Network and external-process automation can fail temporarily. A retry strategy should be deliberate rather than an unconditional loop.

Think about:

- which errors are transient;
- which errors are permanent;
- maximum attempts;
- timeout per attempt;
- total operation deadline;
- delay between attempts;
- whether repeating the operation is safe.

Do not retry blindly. Repeating a non-idempotent operation can make an incident worse.

---

# 33. Processes and exit codes

Automation tools communicate success or failure through process exit status.

A Python program can terminate with an explicit status using `sys.exit()`.

```python
import sys

if not configuration_is_valid:
    print("Invalid configuration")
    sys.exit(1)
```

Use exit codes consistently so CI systems, schedulers, and shell scripts can detect failures.

Do not confuse:

```text
printed error message
```

with:

```text
non-zero process exit status
```

A program can print an error and still exit successfully unless its control flow sets a failure status.

---

# 34. Working with cloud and infrastructure SDKs

Cloud providers and infrastructure platforms commonly expose Python SDKs.

The correct workflow is:

```text
identify the platform
       ↓
read official Python SDK documentation
       ↓
authenticate using the supported mechanism
       ↓
call the documented API
       ↓
validate the response
       ↓
handle errors
       ↓
log useful non-sensitive information
```

Do not memorize provider-specific method names from an unrelated tutorial. SDK APIs change and differ between providers.

For AWS, for example, verify the current official SDK documentation before writing Boto3 code.

---

# 35. Kubernetes automation with Python

Python can be used to interact programmatically with Kubernetes through the official Kubernetes Python client.

Learn the general concepts first:

- Kubernetes API
- resources
- namespaces
- authentication/configuration
- listing resources
- creating/updating resources
- watching resources
- handling API errors

A basic Kubernetes automation program conceptually looks like:

```text
load Kubernetes configuration
        ↓
create API client
        ↓
request resource data
        ↓
validate response
        ↓
perform required action
        ↓
handle errors and report result
```

Use the official Kubernetes Python client documentation for the exact API classes and current examples. Do not hard-code API fields or resource behavior from memory.

---

# 36. Remote automation

Remote automation can be performed through platform-specific tools, SSH-related libraries, APIs, or configuration-management systems.

The Python fundamentals remain the same:

```text
collect input
→ validate
→ connect
→ execute action
→ inspect result
→ report
```

When a project uses a third-party automation library, its official documentation becomes the source of truth for its API and security model.

---

# 37. Webhooks and event-driven automation

A webhook commonly follows this pattern:

```text
external system
      ↓
HTTP request
      ↓
Python service
      ↓
validate event
      ↓
perform automation
      ↓
return appropriate response
```

Important concerns include:

- authentication/signature verification;
- input validation;
- duplicate events;
- retries from the sender;
- idempotency;
- timeouts;
- logging;
- error handling;
- secret management.

Never trust webhook payloads simply because they came from an expected URL.

---

# 38. Python project structure

As scripts grow, organize them as projects rather than one giant file.

A simple structure can look like:

```text
automation-tool/
├── README.md
├── pyproject.toml
├── src/
│   └── automation_tool/
│       ├── __init__.py
│       ├── cli.py
│       ├── config.py
│       └── checks.py
└── tests/
    ├── test_config.py
    └── test_checks.py
```

The exact packaging structure depends on the project requirements. The important lesson is separation of concerns and reproducibility.

---

# 39. Virtual environments

Python's `venv` module creates lightweight virtual environments with an environment-specific set of installed packages.

Create one with:

```bash
python -m venv .venv
```

The exact activation command depends on the operating system and shell. Follow the Python documentation for the environment you are using.

Virtual environments should generally not be committed to Git. They are disposable and should be recreated from project dependency information.

Understand the difference between:

```text
Python interpreter
virtual environment
installed package
project dependency
```

These are related but not the same thing.

---

# 40. Dependency management

Before adding a package, ask:

1. Is the functionality already available in the standard library?
2. Is an external package actually required?
3. Is the package maintained?
4. What license does it use?
5. What versions does it support?
6. How will dependencies be reproduced in CI?
7. How will security updates be handled?

Do not copy dependency versions from an unrelated project without understanding why they were selected.

---

# 41. Code quality for DevOps scripts

A good automation script should be:

- readable;
- predictable;
- testable;
- observable;
- explicit about failures;
- safe to rerun when appropriate;
- careful with secrets;
- small enough to understand;
- structured so logic can be tested independently.

Prefer:

```python
def validate_server(server):
    ...


def deploy(server):
    ...


def main():
    ...
```

over a 500-line function containing every operation.

---

# 42. Common mistakes

## Mistake 1: hard-coding secrets

Bad:

```python
TOKEN = "real-secret-value"
```

Use the supported secret/configuration mechanism instead.

## Mistake 2: swallowing exceptions

Bad:

```python
try:
    deploy()
except Exception:
    pass
```

This can hide a real failure.

## Mistake 3: assuming command success

Always consider the process return code.

## Mistake 4: parsing structured data with fragile string operations

Use the correct parser for JSON, CSV, or another documented format.

## Mistake 5: giant scripts

Split reusable logic into functions and modules.

## Mistake 6: no tests

Test important transformation and decision logic.

## Mistake 7: unsafe retries

Retry only when the failure is appropriate for retry and the operation can safely be repeated.

## Mistake 8: logging sensitive information

Logs often have broad access and long retention. Treat them accordingly.

## Mistake 9: ignoring input validation

Automation should fail early when required inputs are invalid.

## Mistake 10: using a third-party package without checking its documentation

External APIs are not interchangeable. Verify the exact library and version used by your project.

---

# 43. Practical projects

Work through these in order.

## Project 1 — File inventory

Build a CLI tool that:

- accepts a directory;
- finds files;
- reports file names and sizes;
- handles a missing directory cleanly;
- logs its major operations;
- has tests for its pure logic.

Skills:

```text
Path
CLI arguments
loops
functions
exceptions
logging
testing
```

## Project 2 — Configuration validator

Build a tool that reads JSON configuration and validates required fields.

Requirements:

- clear validation errors;
- no secret values in logs;
- non-zero exit status on invalid configuration;
- tests for valid and invalid inputs.

## Project 3 — Log analyzer

Build a program that reads a log file and reports counts of selected event types.

Skills:

```text
files
strings
dictionaries
functions
regex where appropriate
testing
```

## Project 4 — CLI health checker

Build a command-line tool that receives service information, performs a documented check, and returns a useful success/failure status.

Include:

- arguments;
- timeout handling;
- logging;
- error handling;
- tests for decision logic.

## Project 5 — REST API automation

Build a small tool that calls a documented API, validates the response, extracts required data, and reports failures clearly.

Do not hard-code credentials.

## Project 6 — Infrastructure inventory

Use an official SDK or API client for a platform you are learning.

The tool should:

- authenticate using the documented mechanism;
- retrieve inventory;
- normalize the data;
- output a useful report;
- handle API failures;
- avoid logging sensitive data.

## Project 7 — Kubernetes resource report

Using the official Kubernetes Python client, build a read-only reporting tool that retrieves documented resource information and produces a human-readable report.

Start with read operations before attempting mutations.

---

# 44. Exercises

After each section, create at least one small program without copying the example.

### Fundamentals

1. Store an environment name and print a deployment message.
2. Convert a numeric string into an integer and handle invalid input.
3. Determine whether a status code represents a successful response.

### Collections

1. Count duplicate service names.
2. Convert a list of environments into unique values.
3. Build a dictionary representing a server.

### Files

1. Read a list of servers from a file.
2. Ignore blank lines.
3. Write a summary file.

### CLI

1. Accept a filename argument.
2. Add a verbose flag.
3. Return a failure status when the file does not exist.

### Subprocess

1. Run a harmless local command.
2. capture its output;
3. inspect its return code;
4. handle a failed command.

### APIs

1. Call a documented public API.
2. inspect the status code.
3. parse JSON.
4. handle an unsuccessful response.

### Testing

Write tests before changing a function that transforms configuration data.

---

# 45. Interview and troubleshooting questions

You should be able to explain these without memorizing a scripted answer.

1. What is the difference between a list, tuple, set, and dictionary?
2. Why is `dict.get()` useful?
3. What is the difference between `==` and `is`?
4. What is the difference between `append()` and `extend()`?
5. Why use functions in automation scripts?
6. What is an exception?
7. Why should exceptions usually be caught specifically?
8. Why use `with open(...)`?
9. What problem does `pathlib` solve?
10. Why use environment variables?
11. What is JSON serialization?
12. Why use `argparse`?
13. What does a subprocess return code tell you?
14. Why use `subprocess.run()`?
15. What is logging and why is it better than scattered prints for serious automation?
16. What should you consider before retrying an API call?
17. What is idempotency?
18. How should secrets be handled?
19. Why use a virtual environment?
20. What should be tested in an automation script?
21. Why should configuration be validated before an action is performed?
22. What is the difference between authentication and authorization?
23. Why should a webhook payload be validated?
24. Why should API-specific behavior be verified against official documentation?
25. How would you design a Python script that can safely be run from CI?

---

# 46. What a DevOps engineer should be able to build

Before moving deeper into Complete Python, you should be comfortable building a small tool that can:

```text
accept configuration
      ↓
validate input
      ↓
read/write files
      ↓
call an external command or API
      ↓
parse structured data
      ↓
handle failures
      ↓
log useful information
      ↓
return an appropriate exit status
      ↓
be tested
```

If you can do that without turning every script into one large function, you have a strong practical Python foundation for DevOps.

---

# 47. Topics to continue in Complete Python

A DevOps engineer can postpone some deeper language topics until the fundamentals above are comfortable.

The continuation path will cover these in greater depth:

- object-oriented design;
- classes and inheritance;
- multiple inheritance;
- special methods;
- iterators and the iterator protocol;
- generators in depth;
- decorators in depth;
- context managers in depth;
- descriptors;
- namespaces and advanced scope behavior;
- advanced function signatures;
- type annotations and typing tools;
- dataclasses;
- protocols and structural typing;
- asynchronous programming;
- threading and multiprocessing in depth;
- advanced packaging;
- module/package architecture;
- performance analysis;
- profiling;
- advanced testing patterns;
- advanced standard-library modules;
- deeper Python internals.

These are not “unimportant” topics. They are simply placed after the DevOps-focused foundation so learners can become productive sooner.

---

# 48. Suggested progression

```text
Python basics
    ↓
conditions + loops
    ↓
functions
    ↓
collections
    ↓
files + pathlib
    ↓
JSON + configuration
    ↓
CLI programs
    ↓
exceptions
    ↓
subprocess
    ↓
logging
    ↓
testing
    ↓
HTTP/API automation
    ↓
cloud/platform SDKs
    ↓
Kubernetes automation
    ↓
production-quality projects
    ↓
Complete Python
```

Do not skip fundamentals simply because the final goal is DevOps. Automation is software, and weak Python fundamentals usually become operational problems later.

---

# 49. Source-grounded learning notes

This curriculum is synthesized from authoritative Python documentation and DevOps-oriented learning material researched for this repository.

Primary Python documentation used to verify language and standard-library behavior:

- Python Tutorial: https://docs.python.org/3/tutorial/
- Python standard library: https://docs.python.org/3/library/
- `pathlib`: https://docs.python.org/3/library/pathlib.html
- `os`: https://docs.python.org/3/library/os.html
- `subprocess`: https://docs.python.org/3/library/subprocess.html
- `argparse`: https://docs.python.org/3/library/argparse.html
- `logging`: https://docs.python.org/3/library/logging.html
- `unittest`: https://docs.python.org/3/library/unittest.html
- `venv`: https://docs.python.org/3/library/venv.html

DevOps-oriented source used to shape the practical progression:

- https://github.com/iam-veeramalla/python-for-devops

The source repository contains a practical progression through Python fundamentals, collections, file operations, AWS automation, REST APIs, configuration, scheduled automation, and DevOps-oriented projects. This repository reorganizes those ideas into a learner-first curriculum rather than reproducing its day-by-day structure.

For platform-specific behavior, the platform's own official documentation remains the source of truth.

---

# 50. Completion checklist

Before considering Python for DevOps complete, check whether you can honestly say “yes” to each item:

- [ ] I can write and run a Python script.
- [ ] I understand variables and common Python types.
- [ ] I can use conditions and loops.
- [ ] I can work confidently with lists and dictionaries.
- [ ] I can write reusable functions.
- [ ] I can split code into modules.
- [ ] I can read and write files safely.
- [ ] I can use `pathlib` for filesystem paths.
- [ ] I understand environment variables.
- [ ] I can parse and generate JSON.
- [ ] I can build a basic CLI with `argparse`.
- [ ] I can execute an external process with `subprocess`.
- [ ] I understand process exit codes.
- [ ] I can handle expected exceptions.
- [ ] I can use logging appropriately.
- [ ] I can write basic unit tests.
- [ ] I understand HTTP request/response concepts.
- [ ] I can consume a documented API.
- [ ] I understand basic authentication and secret-handling concerns.
- [ ] I understand idempotency and safe retry concepts.
- [ ] I can create and use a virtual environment.
- [ ] I can structure a small Python automation project.
- [ ] I can explain why a script failed by reading its traceback/logs.
- [ ] I can build a small end-to-end automation project.
- [ ] I know when to consult official documentation instead of guessing.

When these are comfortable, move to **Complete Python** for the deeper language and software-engineering topics.

---

## The goal

The goal is not to become someone who knows every Python feature.

The goal is to become a DevOps engineer who can look at an automation problem and confidently think:

```text
What is the input?
What is the desired state?
What needs to be validated?
What Python data structures fit the problem?
What external system must I call?
What can fail?
Can the operation be safely retried?
What should be logged?
What should be tested?
What should the program return?
```

That mindset is more valuable than memorizing syntax.