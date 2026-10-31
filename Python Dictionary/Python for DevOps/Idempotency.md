# Idempotency

Idempotency is a core property of reliable automation: an operation can be executed repeatedly without repeatedly producing an unintended change in the resulting state.

For DevOps automation, the useful question is usually not **“Did my script run?”** but **“After running it again, is the system still in the desired state?”**

> **Verified fact:** RFC 9110 defines an idempotent request method as one where multiple identical requests have the same intended effect as a single such request, although the response can differ.
>
> **Repository design decision:** This chapter applies the same state-oriented thinking to Python automation such as filesystem and configuration tasks. A Python script is not automatically idempotent merely because it is safe to run twice.

## 1. Mental model

Think in terms of **desired state** rather than a sequence of mutations.

```text
Current state → inspect → compare with desired state → change only what is needed
```

A useful automation loop is:

1. Read the current state.
2. Define the desired state.
3. Determine the difference.
4. Apply the smallest required change.
5. Verify the resulting state.
6. Make the same run safe to repeat.

Idempotency does **not** mean “nothing ever changes.” The first run may make a change. The important property is that repeating the operation does not keep accumulating the same effect.

## 2. Idempotent vs non-idempotent automation

Consider a script that must ensure a directory exists.

A direct filesystem creation operation can fail if the directory already exists. Python's `pathlib.Path.mkdir()` supports `exist_ok=True`, which allows an existing directory to be treated as an acceptable final state.

```python
from pathlib import Path

logs = Path("logs")
logs.mkdir(exist_ok=True)
```

The intent is:

```text
Desired state: logs/ exists as a directory

Run 1 → create it if necessary
Run 2 → it already exists; no new directory is accumulated
Run 3 → same desired state
```

By contrast, blindly appending a line every time is not idempotent:

```python
from pathlib import Path

Path("config.txt").open("a", encoding="utf-8").write("enabled=true\n")
```

Repeated execution can produce duplicate lines. If the desired state is “the setting appears exactly once,” the operation needs to inspect and reconcile the existing content instead.

## 3. Check before change

A simple pattern is:

```python
from pathlib import Path

path = Path("service.conf")
line = "enabled=true"

if path.exists():
    content = path.read_text(encoding="utf-8")
else:
    content = ""

if line not in content.splitlines():
    with path.open("a", encoding="utf-8") as file:
        file.write(line + "\n")
```

The important design idea is the **state check**, not this exact implementation.

For configuration automation, make the desired representation explicit and compare it with what is already present. Avoid relying on a loose substring test when the configuration format has meaningful structure.

## 4. Ensure desired state

“Ensure” operations are easier to reason about than “always perform” operations.

Examples:

| Task | Better automation intent |
|---|---|
| Directory setup | Ensure the directory exists |
| File permissions | Ensure the required permissions are present |
| Configuration | Ensure the desired setting is represented correctly |
| Service configuration | Ensure the intended configuration is applied |
| Resource creation | Look up existing state before creating another resource |
| Cleanup | Ensure unwanted state is absent |

This is a repository learning pattern, not a claim that every external system exposes an idempotent API.

## 5. Make absence idempotent too

Python's `Path.unlink()` supports `missing_ok=True` for treating a missing file as an acceptable result of the removal operation.

```python
from pathlib import Path

Path("temporary.txt").unlink(missing_ok=True)
```

The desired state here is **“temporary.txt does not exist.”**

This is useful for cleanup code where the resource may already have been removed by an earlier run.

When working with APIs or external systems, do not assume that a similar option exists. Verify the API's documented behavior before relying on it.

## 6. Synchronize instead of append

Suppose a generated file should contain exactly this content:

```text
mode=production
region=ap-south-1
```

An append-based implementation can accumulate duplicates. A state-oriented implementation can construct the complete desired representation and compare it with the current file.

```python
from pathlib import Path

path = Path("settings.conf")
desired = "mode=production\nregion=ap-south-1\n"

current = path.read_text(encoding="utf-8") if path.exists() else ""

if current != desired:
    path.write_text(desired, encoding="utf-8")
```

This demonstrates a useful pattern for generated configuration: **one desired representation, one comparison, one update when the states differ.**

For more complex formats, parse and validate the configuration instead of treating structured data as arbitrary text.

## 7. Idempotency and retries

Automation commonly needs retries because a remote operation can fail or a process can lose its connection after the operation has been sent.

Idempotent operations reduce the risk that a retry unintentionally accumulates the same change again.

However:

> **Verified fact:** Idempotency is a property of the intended effect of an operation; it does not mean that every execution returns the same response or that failures cannot occur.

> **Repository design decision:** For Python automation, identify the boundary at which a retry can repeat a state-changing action and ask whether that action is safe to repeat.

Do not “add retries” first and reason about side effects later. Design the operation and its retry behavior together.

## 8. HTTP idempotency is a protocol concept

HTTP provides a precise specification for idempotent methods. RFC 9110 describes methods such as `PUT` and `DELETE` as idempotent, while `POST` is not defined as idempotent by that method property.

This does **not** mean that every real-world API call using one of these methods is automatically safe for every business operation. The protocol definition concerns the intended effect of repeating the same request, and application-specific side effects can still matter.

When writing Python HTTP automation, consult the API's own documentation for its retry and idempotency guarantees rather than inferring them from a method name alone.

Reference: https://www.rfc-editor.org/rfc/rfc9110.html

## 9. Idempotency keys

Some APIs document an **idempotency key** mechanism so a client can associate retries of a request with the same logical operation.

The exact header name, scope, retention period, conflict behavior, and supported operations are API-specific. Do not invent these details or assume that all APIs implement them.

For a particular service:

1. Read the official API documentation.
2. Confirm whether idempotency keys are supported.
3. Confirm the exact request field/header and semantics.
4. Understand what happens when the same key is reused.
5. Test the documented retry behavior.

## 10. Resource creation: look before creating

A common automation mistake is treating “create” as the only operation needed.

Conceptually:

```text
Bad:
    create resource
    create resource
    create resource

Better:
    inspect current state
    if desired resource already exists:
        verify or reconcile it
    else:
        create it
```

Whether this is possible depends on the target system. Some APIs provide a stable identifier or lookup operation; others provide their own create-or-update semantics. Verify the target API instead of assuming one exists.

## 11. Separate planning from applying

A dry-run or plan mode can make idempotent automation easier to inspect.

A useful design is:

```text
inspect current state
        ↓
calculate desired changes
        ↓
show planned changes
        ↓
apply changes only when requested
```

The exact implementation is a repository design decision. The important property is that planning should not unexpectedly mutate the target system.

## 12. Verify after the change

A robust automation script should know what state it intended to produce.

For example:

```python
from pathlib import Path

path = Path("service.conf")
desired = "enabled=true\n"

current = path.read_text(encoding="utf-8") if path.exists() else ""

if current != desired:
    path.write_text(desired, encoding="utf-8")

assert path.read_text(encoding="utf-8") == desired
```

The assertion is useful in a small demonstration or test. Production automation may instead use explicit validation and an appropriate error-handling strategy.

## 13. Test idempotency explicitly

A practical test is to run the operation twice and inspect the resulting state.

```text
Initial state
    ↓
Run automation
    ↓
Capture state A
    ↓
Run automation again
    ↓
Capture state B
    ↓
Compare A and B
```

For a well-designed idempotent operation, the second run should not introduce an additional unintended change.

For filesystem examples, useful checks include:

- Does a required directory remain a single directory?
- Does generated configuration remain identical?
- Are unwanted files still absent?
- Are repeated cleanup operations harmless?
- Does the second run report no unnecessary changes?

## 14. Common mistakes

### Mistake 1: Always append

Appending configuration on every run often creates duplicate state.

**Better:** parse or compare the desired representation and update only when needed.

### Mistake 2: Treating “script succeeded” as “state is correct”

A command can exit successfully while leaving the system in an unintended state.

**Better:** verify the state that matters.

### Mistake 3: Assuming every API supports idempotency

Different APIs expose different guarantees.

**Better:** read the official API documentation.

### Mistake 4: Adding retries around non-idempotent operations

A retry can repeat a state-changing request after an uncertain failure.

**Better:** understand the operation's side effects and documented retry semantics before adding automatic retries.

### Mistake 5: Using a weak equality check for structured configuration

Text equality can be inappropriate when ordering, formatting, or defaults are not semantically meaningful.

**Better:** parse the configuration and compare the relevant semantic state when appropriate.

### Mistake 6: Hiding destructive behavior behind “ensure” wording

An operation that deletes or overwrites data can still be repeatable while being unsafe.

**Better:** distinguish **idempotent** from **safe**. They are different properties.

## 15. Practical DevOps exercise

Build a Python script that manages a small generated configuration file.

Requirements:

1. Define a desired configuration in Python.
2. Create the file if it does not exist.
3. Detect when the existing content differs.
4. Update the file only when necessary.
5. Run the script twice.
6. Demonstrate that the second run does not duplicate content.
7. Add a test that verifies the final file content.
8. Add a case where the existing configuration is intentionally different and verify that the script reconciles it.

### Extension
Add a `--dry-run` option and make the script report what it would change without modifying the file.

## 16. Knowledge checks

1. What is the difference between idempotency and safety?
2. Why can repeated `append` operations be non-idempotent?
3. What does “ensure desired state” mean?
4. Why should retry behavior be considered together with side effects?
5. Does an HTTP method being idempotent guarantee that every application-specific operation is safe? Why or why not?
6. How would you test whether a Python automation script is idempotent?
7. When should structured configuration be parsed rather than compared as raw text?

## 17. Key takeaways

- Design automation around **desired state**.
- Inspect before changing when the target system requires it.
- Avoid accumulating duplicate state.
- Make repeated execution a deliberate test case.
- Treat retries and side effects as one design problem.
- Do not infer API guarantees; verify them from authoritative documentation.
- Idempotency and safety are related but distinct properties.

## Verified facts vs repository design

### Verified facts

- Python's `pathlib.Path.mkdir()` provides `exist_ok` for directory creation behavior.
- Python's `pathlib.Path.unlink()` provides `missing_ok` for handling a missing path.
- RFC 9110 defines idempotent HTTP methods in terms of the intended effect of repeated identical requests.
- RFC 9110 identifies `PUT` and `DELETE` as idempotent methods and does not classify `POST` as idempotent by this method property.

### Repository design decisions

- The examples use “ensure desired state” as the primary teaching model.
- The two-run test is recommended as a practical learning exercise.
- The examples intentionally focus on standard-library Python and generic automation patterns rather than a particular cloud provider or API.

### Demonstrations

- The filesystem examples are teaching demonstrations. They should not be interpreted as universal production-safe file-update algorithms for every environment.

## Authoritative references

- Python `pathlib`: https://docs.python.org/3/library/pathlib.html
- Python `shutil`: https://docs.python.org/3/library/shutil.html
- Python `filecmp`: https://docs.python.org/3/library/filecmp.html
- Python standard library: https://docs.python.org/3/library/
- RFC 9110 — HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110.html
