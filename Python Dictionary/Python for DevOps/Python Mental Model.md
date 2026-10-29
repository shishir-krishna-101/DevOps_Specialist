# Python Mental Model

Python is a general-purpose programming language. For DevOps work, it is particularly useful for connecting systems: a script can read configuration, inspect files, call an API, execute another process, transform data, and report the result.

> **Verified fact:** Python's standard library provides interfaces for common operating-system tasks, filesystem paths, subprocesses, command-line parsing, logging, JSON, testing, and other automation work.
>
> **Repository design decision:** This topic uses those capabilities to build a practical mental model for DevOps automation rather than teaching Python as an isolated programming language.

---

## 1. The Core Automation Mental Model

A useful way to think about a DevOps automation program is:

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

The exact stages vary by task, but the separation is valuable because each stage answers a different question:

| Stage | Question |
|---|---|
| Input | Where does the information come from? |
| Parse | What does the input mean? |
| Validate | Is it acceptable and complete? |
| Transform | What representation does the next step need? |
| Action | What change or operation should occur? |
| Check | Did the operation succeed? |
| Report | What should the operator or next system know? |

**Repository design decision:** Keeping these stages conceptually separate makes automation easier to test, troubleshoot, and extend.

---

## 2. Python as a Glue Language

DevOps automation often sits between systems rather than implementing one large business domain.

For example, one program might:

```text
configuration
     ↓
Python
 ↙   ↓   ↘
files  API  process
     ↓
result / report
```

Python can provide the control flow while standard-library or project-specific libraries handle the individual boundaries.

Typical boundaries include:

- filesystem
- environment variables
- command-line arguments
- subprocesses
- HTTP APIs
- JSON data
- databases
- cloud or infrastructure SDKs
- Kubernetes APIs

The important skill is not memorizing every library. It is recognizing the boundary you need to cross and then using the authoritative documentation for that interface.

---

## 3. Objects, Names, and Values

A Python variable name refers to an object.

```python
server_count = 3
service_name = "api"
is_healthy = True
```

The names above refer to objects representing an integer, a string, and a boolean value.

A useful beginner mental model is:

```text
name ─────→ object
```

This becomes important when working with mutable objects, function arguments, aliases, and configuration structures.

Reference: https://docs.python.org/3/tutorial/introduction.html

---

## 4. Common Built-in Types for Automation

You should become comfortable with these built-in types early:

| Type | Example | Typical automation use |
|---|---|---|
| `int` | `3` | counts, ports, retry limits |
| `float` | `2.5` | measurements and ratios |
| `str` | `"production"` | names, paths, API data |
| `bool` | `True` | state checks |
| `None` | `None` | absence of a value |
| `list` | `["web", "db"]` | ordered collections |
| `tuple` | `("prod", "web")` | fixed sequences |
| `set` | `{ "dev", "prod" }` | unique values |
| `dict` | `{ "env": "prod" }` | structured key/value data |

Check a value while learning:

```python
value = "production"
print(type(value))
```

Reference: https://docs.python.org/3/library/stdtypes.html

---

## 5. Mutable and Immutable Objects

Some Python objects can be changed after creation, while others cannot.

Common mutable built-in containers include:

- `list`
- `dict`
- `set`

Common immutable objects include:

- `str`
- `tuple`
- numbers

Example:

```python
servers = ["web-1", "web-2"]
servers.append("web-3")
```

The list was modified.

With a string:

```python
name = "web"
name = name + "-api"
```

The binding is changed to a new string value; the original string itself is not modified.

Understanding mutability becomes especially important when functions receive collections or when several names refer to the same object.

Reference: https://docs.python.org/3/reference/datamodel.html

---

## 6. Expressions and Operators

Automation code frequently makes decisions using comparisons and boolean expressions.

```python
retries = 2
limit = 3

if retries < limit and limit > 0:
    print("retry is still allowed")
```

Become comfortable with:

- arithmetic operators
- comparison operators
- boolean operators
- membership operators
- identity checks

### `==` versus `is`

`==` compares values. `is` checks object identity.

Use `==` for ordinary value comparison:

```python
if environment == "production":
    print("production environment")
```

Use `is` for identity-sensitive checks, commonly when testing for `None`:

```python
result = None

if result is None:
    print("no result")
```

Do not use `is` as a general replacement for `==`.

Reference: https://docs.python.org/3/reference/expressions.html#comparisons

---

## 7. Input Should Have a Clear Boundary

Automation becomes easier to reason about when external input enters the program through an identifiable boundary.

Examples:

```text
command-line arguments
        ↓
     Python

environment variables
        ↓
     Python

configuration file
        ↓
     Python

API response
        ↓
     Python
```

Once the input enters the program, parse and validate it before relying on it.

**Repository design decision:** The repository treats parsing and validation as explicit stages instead of mixing untrusted or loosely structured input throughout the rest of the program.

---

## 8. Parse, Validate, Then Act

Consider a deployment script that receives an environment name.

A safer conceptual flow is:

```text
raw input
   ↓
parse
   ↓
validate
   ↓
construct operation
   ↓
perform operation
```

Example:

```python
def require_environment(value):
    if not value:
        raise ValueError("environment is required")
    return value


environment = require_environment("production")
print(environment)
```

The validation rule should reflect the real requirements of the system. Do not invent validation rules simply because they seem convenient.

---

## 9. Functions as Units of Work

Functions allow automation logic to be split into understandable units.

```python
def is_success(status_code):
    return 200 <= status_code < 300
```

A larger workflow can then be described through functions such as:

```text
read_config()
validate_config()
build_command()
run_command()
check_result()
report_result()
```

This is a repository design pattern, not a Python requirement.

Good functions generally have a clear purpose and predictable inputs and outputs.

Reference: https://docs.python.org/3/tutorial/controlflow.html#defining-functions

---

## 10. Separate Policy from Mechanism

A useful DevOps design distinction is:

```text
Policy
  ↓
What should happen?

Mechanism
  ↓
How is it performed?
```

For example:

```python
def should_retry(attempt, limit):
    return attempt < limit
```

can represent a policy, while another function can perform the actual operation.

Keeping the two concerns separate makes the policy easier to test without invoking the real infrastructure operation.

**Repository design decision:** This separation is encouraged throughout the Python-for-DevOps track because infrastructure actions are often harder and riskier to test than pure decision logic.

---

## 11. Check Results Explicitly

A script should not assume that an action succeeded merely because Python reached the next line.

For an operation that returns a result, inspect that result according to the API or tool's documented behavior.

Conceptually:

```python
result = perform_operation()

if not operation_succeeded(result):
    raise RuntimeError("operation failed")
```

The exact success condition depends on the system being automated. Do not invent exit codes, response fields, or success rules.

---

## 12. Fail Clearly

Automation failures should be observable and actionable.

Python exceptions provide a structured way for failures to propagate.

```python
def require_name(name):
    if not name:
        raise ValueError("name is required")
```

Catch an exception when the program can meaningfully handle it or add useful context. Do not catch every exception merely to keep the program running.

Reference: https://docs.python.org/3/tutorial/errors.html

---

## 13. Logging Is Different from Printing

`print()` is useful for simple scripts and learning.

For reusable automation, Python's `logging` module provides a standard logging facility with concepts such as loggers, levels, handlers, and formatters.

Example:

```python
import logging

logger = logging.getLogger(__name__)
logger.info("starting deployment")
```

Use the logging documentation and the surrounding application's logging configuration when designing production tooling.

Never log secrets or sensitive values merely because they are available to the program.

Reference: https://docs.python.org/3/library/logging.html

---

## 14. A Small End-to-End Example

The following example demonstrates the mental model without connecting to real infrastructure:

```python
from pathlib import Path


def read_service_name(path):
    value = path.read_text().strip()
    if not value:
        raise ValueError("service name is empty")
    return value


def build_message(service):
    return f"Checking service: {service}"


def main():
    config = Path("service.txt")
    service = read_service_name(config)
    message = build_message(service)
    print(message)


if __name__ == "__main__":
    main()
```

The flow is:

```text
service.txt
    ↓
read_service_name()
    ↓
validate
    ↓
build_message()
    ↓
report
```

The example intentionally stops before performing a real infrastructure action.

**Demonstration:** This example is a teaching exercise created for this repository. It does not represent a particular production system or DevOps platform.

---

## 15. Common Mistakes

### Mistake 1: Putting everything in one function

A huge function makes testing and troubleshooting harder.

Prefer clear units of work when the program becomes non-trivial.

### Mistake 2: Mixing input parsing with infrastructure actions

If input validation and destructive actions are tightly coupled, testing becomes harder and mistakes become more costly.

### Mistake 3: Assuming success

Inspect the documented result of the operation you invoked.

### Mistake 4: Catching every exception

Broad exception handling can hide real failures.

### Mistake 5: Logging sensitive data

Credentials, tokens, private keys, and other sensitive values should not be printed simply for debugging convenience.

### Mistake 6: Treating every external input as trusted

Configuration files, environment variables, command-line arguments, API responses, and command output all need appropriate validation before use.

### Mistake 7: Replacing documentation with memory

When an automation task depends on a tool or API, verify its actual interface from authoritative documentation instead of guessing flags, fields, response shapes, or behavior.

---

## 16. DevOps Practice Exercises

### Exercise 1 — Configuration reader

Write a function that:

1. accepts a `Path`;
2. verifies the file exists;
3. reads its text;
4. rejects an empty value;
5. returns the normalized value.

### Exercise 2 — Health decision

Write a pure function that receives a documented status value and returns whether it represents success according to a rule you explicitly define.

### Exercise 3 — Workflow separation

Take a small automation script and split it into:

```text
input
validation
transformation
action
result checking
reporting
```

### Exercise 4 — Failure path

Add an intentional validation failure and observe the traceback. Explain:

- exception type
- exception message
- location where it was raised
- how the exception propagated

### Exercise 5 — Logging

Replace diagnostic `print()` calls in a small script with `logging` calls. Experiment with log levels and explain why each message uses its chosen level.

---

## 17. Knowledge Check

You should be able to answer these without looking at the notes:

1. Why is Python useful for DevOps automation?
2. What is the difference between a name and an object?
3. Which common built-in containers are mutable?
4. What is the difference between `==` and `is`?
5. Why should external input have a clear boundary?
6. Why separate parsing and validation from an infrastructure action?
7. Why should automation check the documented result of an operation?
8. When should an exception be caught?
9. Why is `logging` useful for reusable automation?
10. Why should production automation avoid guessing external tool or API behavior?

---

## 18. Completion Criteria

You have completed this topic when you can:

- explain Python's role in DevOps automation;
- describe the input → validate → act → check → report flow;
- distinguish names, objects, and values at a practical level;
- use common built-in types confidently;
- distinguish `==` from `is`;
- structure a small automation program into functions;
- validate input before acting on it;
- handle expected failures without hiding unexpected ones;
- explain why result checking matters;
- use logging appropriately for reusable automation;
- identify where authoritative documentation is required before automating an external system.

---

## Authoritative References

- Python Tutorial: https://docs.python.org/3/tutorial/
- Python Tutorial — Introduction: https://docs.python.org/3/tutorial/introduction.html
- Python Tutorial — Control Flow: https://docs.python.org/3/tutorial/controlflow.html
- Python Tutorial — Errors and Exceptions: https://docs.python.org/3/tutorial/errors.html
- Python Standard Types: https://docs.python.org/3/library/stdtypes.html
- Python Data Model: https://docs.python.org/3/reference/datamodel.html
- Python Expressions: https://docs.python.org/3/reference/expressions.html
- Python Logging: https://docs.python.org/3/library/logging.html
