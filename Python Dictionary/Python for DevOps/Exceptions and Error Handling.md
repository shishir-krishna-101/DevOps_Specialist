# Exceptions and Error Handling

Exceptions provide Python's mechanism for reporting and handling conditions that interrupt normal execution. In DevOps automation, good exception handling is important because a script may interact with files, processes, networks, APIs, or infrastructure where failures are expected possibilities rather than unusual surprises.

## 1. Mental Model

Think of an exception workflow as:

```text
operation
   ↓
success → continue
   ↓
failure → exception
   ↓
appropriate boundary
   ↓
handle / report / propagate
```

**Verified fact:** Python uses exceptions to report errors and other exceptional conditions during execution.

**Repository design decision:** The examples focus on automation boundaries where failures should remain visible, diagnosable, and actionable.

## 2. What Is an Exception?

An exception is an event that interrupts the normal flow of program execution.

For example:

```python
value = int("not-a-number")
```

This raises an exception instead of producing an integer.

The important automation lesson is that external operations can fail and the program needs an intentional policy for what happens next.

## 3. Raising an Exception

Code can explicitly raise an exception with `raise`:

```python
def require_region(config):
    if "region" not in config:
        raise ValueError("region is required")

    return config["region"]
```

Raising an exception is useful when a function cannot satisfy its contract or when continuing would produce an invalid result.

## 4. `try` and `except`

Use `try` to identify code whose exceptions you want to handle:

```python
try:
    value = int("42")
except ValueError:
    print("invalid integer")
```

The `except` block handles the selected exception type.

Prefer catching a specific exception when the recovery or reporting behavior depends on that exception.

## 5. Catch the Narrowest Useful Exception

Prefer:

```python
try:
    value = int(raw_value)
except ValueError:
    value = 0
```

over an unnecessarily broad handler:

```python
try:
    value = int(raw_value)
except Exception:
    value = 0
```

A broad handler can accidentally treat unrelated programming or operational failures as if they were the expected input problem.

The correct exception boundary depends on the operation and recovery policy.

## 6. Multiple Exception Types

A handler can catch multiple related exception types:

```python
try:
    load_configuration()
except (OSError, ValueError):
    print("configuration could not be loaded")
```

Only group exceptions together when the handling behavior is genuinely the same.

If different failures require different actions, use separate handlers.

## 7. `else`

A `try` statement can have an `else` clause:

```python
try:
    value = int(raw_value)
except ValueError:
    print("invalid value")
else:
    print(f"using {value}")
```

The `else` suite runs when the `try` suite completes without raising an exception.

This can keep success-path code separate from exception-handling code.

## 8. `finally`

A `finally` clause is used for cleanup that should be attempted regardless of whether an exception occurred:

```python
resource = acquire_resource()
try:
    use_resource(resource)
finally:
    release_resource(resource)
```

For resources that support context managers, prefer `with` when appropriate. Context managers are covered in **Complete Python**.

## 9. Exception Propagation

If a function does not handle an exception, it can propagate to its caller:

```python
def parse_port(value):
    return int(value)


def configure_service(value):
    port = parse_port(value)
    return port
```

If `int()` raises `ValueError`, the exception can move through the call stack until an appropriate handler is reached.

This is often preferable to catching an exception at a low level when that layer does not know how the failure should be handled.

## 10. Handling at the Right Boundary

A useful automation structure is:

```text
low-level operation
       ↓
raise meaningful exception
       ↓
workflow boundary
       ↓
log / report / choose exit behavior
```

For example, a function that reads a configuration file may report a filesystem failure, while the top-level CLI workflow decides how to present that failure to the operator.

Do not automatically handle every exception where it occurs.

## 11. Preserving the Original Failure

When converting one exception into another, use exception chaining when the original failure is relevant:

```python
try:
    load_configuration()
except OSError as exc:
    raise RuntimeError("configuration could not be loaded") from exc
```

This preserves the relationship between the higher-level failure and the original exception.

Use this when a higher-level abstraction needs to communicate a more useful application-level failure without losing diagnostic context.

## 12. Custom Exceptions

A project can define its own exception classes:

```python
class ConfigurationError(Exception):
    pass
```

Then:

```python
raise ConfigurationError("missing deployment region")
```

Custom exceptions can make application-specific failure categories clearer.

Keep the hierarchy purposeful rather than creating a separate exception class for every possible message.

## 13. Exception Hierarchies

Python exceptions form class hierarchies. This means a handler can target a particular exception type or a broader parent type when appropriate.

For example:

```python
try:
    operation()
except OSError:
    recover_from_os_failure()
```

A handler for a base exception class can also cover relevant subclasses.

The key design question is whether all exceptions matched by the handler should receive the same treatment.

## 14. `BaseException` vs `Exception`

Most application-level errors derive from `Exception`.

Do not normally use:

```python
except BaseException:
    ...
```

as a general-purpose application error handler.

`BaseException` is higher in the exception hierarchy and includes exceptions that generally should not be swallowed by ordinary application error handling.

Catch the exception category that matches the intended policy.

## 15. Do Not Swallow Failures

Avoid code such as:

```python
try:
    deploy()
except Exception:
    pass
```

This can make an automation run appear successful when the intended operation actually failed.

If a failure is intentionally ignored, document why and make sure ignoring it is part of the desired workflow behavior.

## 16. Logging Exceptions

For operational automation, a failure often needs useful diagnostic context.

For example:

```python
import logging

logger = logging.getLogger(__name__)

try:
    deploy()
except OSError:
    logger.exception("deployment operation failed")
    raise
```

`logger.exception()` is intended for use while handling an exception and includes exception information in the log record.

Do not log secrets, tokens, passwords, or other sensitive values merely to make an error easier to diagnose.

## 17. Re-Raising Exceptions

A handler can perform an action and then re-raise the current exception:

```python
try:
    operation()
except OSError:
    logger.exception("operation failed")
    raise
```

The bare `raise` preserves the current exception being handled.

This is useful when the current layer needs to record context but does not own the final recovery decision.

## 18. Input Validation

Exception handling should not replace validation.

For example:

```python
def validate_environment(environment):
    allowed = {"dev", "staging", "production"}

    if environment not in allowed:
        raise ValueError(f"unsupported environment: {environment}")

    return environment
```

Explicit validation creates a predictable failure boundary.

A caller can then decide whether to report the validation error, request corrected input, or terminate the workflow.

## 19. Filesystem Failures

Filesystem operations can raise exceptions such as `OSError` subclasses.

For example:

```python
from pathlib import Path

path = Path("config.json")

try:
    content = path.read_text(encoding="utf-8")
except OSError as exc:
    raise RuntimeError(f"could not read configuration: {path}") from exc
```

The exact handling policy depends on whether a missing or unreadable file is expected, recoverable, or fatal for the workflow.

## 20. Command Execution Failures

When Python starts external commands, the subprocess API can report failures through exceptions depending on how it is used.

For example, `subprocess.run()` with `check=True` raises `CalledProcessError` when the completed process returns a non-zero exit status.

```python
import subprocess

try:
    subprocess.run(["python", "--version"], check=True)
except subprocess.CalledProcessError as exc:
    print(f"command failed with exit code {exc.returncode}")
```

**Verified fact:** The behavior of `subprocess.run()` and `CalledProcessError` should be checked against the Python version and official documentation used by the project.

Do not assume that every non-zero process result automatically raises an exception; the subprocess API provides explicit options controlling this behavior.

## 21. Network and API Failures

Network automation can fail because of connectivity, timeouts, remote responses, authentication, or application-level errors.

Do not treat every network exception as automatically retryable.

A useful decision process is:

```text
failure
  ↓
what failed?
  ↓
transient or permanent?
  ↓
safe to retry?
  ↓
retry / report / abort
```

Retry policy is an application design decision and should account for idempotency, timeouts, and the specific API behavior involved.

## 22. Avoid Exception-Driven Normal Flow When Simpler Logic Exists

Exceptions are useful for exceptional conditions, but do not use complicated exception handling where a straightforward check communicates the intent better.

For example, when checking whether a known dictionary key exists:

```python
if "region" in config:
    region = config["region"]
```

can be clearer than deliberately triggering and catching `KeyError` when there is no reason to do so.

On the other hand, when an operation naturally reports failure through an exception, handling that exception can be appropriate.

## 23. `KeyError`, `IndexError`, and `TypeError`

Common built-in exceptions are useful signals:

```python
config["region"]       # may raise KeyError
items[10]               # may raise IndexError
"value" + 10           # raises TypeError
```

Handle these according to the actual contract of the operation rather than catching all exceptions indiscriminately.

For configuration access, `dict.get()` can sometimes be preferable when a missing value is an expected condition and a default or explicit follow-up policy is appropriate.

## 24. Exception Messages

Exception messages should help identify what failed without exposing sensitive data.

Prefer:

```python
raise ValueError(f"unsupported environment: {environment}")
```

when `environment` is safe to include.

Avoid messages such as:

```text
API token abc123... is invalid
```

if the value could expose a secret.

Operational diagnostics should be useful without turning logs or error output into a secret-disclosure channel.

## 25. Exceptions at CLI Boundaries

A command-line application often needs a clear final failure behavior.

A useful structure is:

```text
main()
  ↓
parse arguments
  ↓
validate configuration
  ↓
perform workflow
  ↓
handle expected application failures
  ↓
report outcome
```

The CLI boundary can convert internal failures into an appropriate user-facing message and process exit behavior.

Do not hide the original cause merely to produce a short message; retain useful diagnostic information through logging or exception chaining where appropriate.

## 26. A Practical DevOps Example

```python
import logging

logger = logging.getLogger(__name__)


class ConfigurationError(Exception):
    pass


def load_environment(config):
    try:
        value = config["environment"]
    except KeyError as exc:
        raise ConfigurationError("environment is required") from exc

    value = value.strip().lower()

    if value not in {"dev", "staging", "production"}:
        raise ConfigurationError(f"unsupported environment: {value}")

    return value


def run_workflow(config):
    environment = load_environment(config)
    logger.info("validated environment: %s", environment)
    return environment


try:
    run_workflow({"environment": "Production"})
except ConfigurationError:
    logger.exception("configuration validation failed")
    raise
```

This example demonstrates a deliberate boundary:

```text
configuration input
       ↓
validate
       ↓
raise domain-specific error
       ↓
workflow boundary
       ↓
log diagnostic context
       ↓
propagate failure
```

**Repository design decision:** The example's environment names and validation policy are teaching data, not a universal deployment policy.

## 27. Common Mistakes

### Mistake 1: Catching `Exception` everywhere

This can hide programming errors and unrelated operational failures.

### Mistake 2: Using `except: pass`

Silent failure is especially dangerous in automation.

### Mistake 3: Logging and suppressing the failure

Logging an exception does not mean the workflow succeeded. Re-raise when the failure remains fatal.

### Mistake 4: Retrying every exception

Not every failure is transient or safe to retry.

### Mistake 5: Losing exception context

When converting exceptions, use exception chaining when the original cause matters.

### Mistake 6: Logging secrets

Diagnostic output must not expose credentials, tokens, or other sensitive configuration.

### Mistake 7: Handling too low in the call stack

A low-level helper may not know whether the overall workflow should retry, abort, or continue.

### Mistake 8: Returning `None` for every failure

This can blur the distinction between a valid `None` result and an actual failure.

### Mistake 9: Using exceptions as universal validation

Clear validation logic is often easier to understand than deliberately triggering exceptions.

## 28. Practical Exercises

### Exercise 1 — Configuration validation

Write a function that validates a required configuration key and raises a custom exception when it is missing.

### Exercise 2 — File handling

Read a configuration file and distinguish between an expected missing-file condition and an unexpected filesystem failure.

### Exercise 3 — Exception chaining

Catch a low-level exception and raise a higher-level application exception using `raise ... from ...`.

### Exercise 4 — Safe command execution

Run a command with `subprocess.run()` and handle a non-zero exit status without hiding the failure.

### Exercise 5 — Retry classification

Create a small function that receives a failure category and decides whether the workflow should retry. Document which decisions are repository policy rather than Python behavior.

### Exercise 6 — Logging review

Review an automation script and identify exception handlers that might expose secrets or silently suppress failures.

### Exercise 7 — Boundary design

Build a three-layer workflow:

```text
helper → raises
workflow → decides
CLI boundary → reports
```

Explain why each layer owns its particular responsibility.

## 29. Knowledge Check

Before moving on, you should be able to answer:

1. What is an exception?
2. What is the purpose of `try` and `except`?
3. Why is catching specific exception types usually preferable?
4. What are `else` and `finally` used for?
5. Why might an exception be allowed to propagate to a higher-level boundary?
6. What does `raise ... from ...` accomplish?
7. Why should automation avoid silently swallowing exceptions?
8. When might a custom exception be useful?
9. Why is not every failure automatically safe to retry?
10. Why should logs and exception messages avoid exposing secrets?

## 30. Completion Criteria

You are ready to move on when you can:

- explain Python's exception model;
- raise and catch appropriate exception types;
- use `try`, `except`, `else`, and `finally` appropriately;
- allow failures to propagate to a meaningful boundary;
- create useful custom exceptions;
- preserve original causes with exception chaining;
- handle filesystem and subprocess failures intentionally;
- distinguish retryable decisions from generic exception handling;
- avoid swallowing failures and leaking sensitive information;
- design an automation workflow with clear failure boundaries.

## 31. Source Notes

### Verified facts

The Python exception behavior described here is based on the official Python documentation.

### Repository design decisions

The DevOps failure boundaries, example policies, exercises, and operational recommendations are teaching choices for this repository. They are not universal requirements for every automation platform.

### Authoritative references

- Python Tutorial — Errors and Exceptions: https://docs.python.org/3/tutorial/errors.html
- Python Language Reference — Exceptions: https://docs.python.org/3/reference/executionmodel.html#exceptions
- Python Language Reference — The `raise` statement: https://docs.python.org/3/reference/simple_stmts.html#the-raise-statement
- Python Language Reference — The `try` statement: https://docs.python.org/3/reference/compound_stmts.html#the-try-statement
- Python Standard Library — Built-in Exceptions: https://docs.python.org/3/library/exceptions.html
- Python Standard Library — `logging`: https://docs.python.org/3/library/logging.html
- Python Standard Library — `subprocess`: https://docs.python.org/3/library/subprocess.html
