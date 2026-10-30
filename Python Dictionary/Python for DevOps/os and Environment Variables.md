# os and Environment Variables

Python's `os` module provides portable access to operating-system-dependent functionality. For DevOps automation, the most useful parts at this stage are environment variables, the current working directory, process information, filesystem operations, and path interoperability.

> **Source note:** This topic is based on the official Python `os` documentation. The examples are teaching demonstrations created for this repository.

---

## 1. The mental model

Think of the operating-system boundary as an external input to your Python program:

```text
Operating system
      ↓
environment / filesystem / process state
      ↓
Python os module
      ↓
read → validate → act → report
```

This boundary matters because operating-system state is not automatically under your program's control.

A good DevOps script makes important environmental assumptions explicit.

---

## 2. Importing `os`

```python
import os
```

The `os` module contains many operating-system interfaces. You should not try to memorize the entire module.

For day-to-day automation, learn the small set of interfaces that solve the problem you actually have.

---

## 3. Environment variables

Environment variables are part of the process environment and are exposed through `os.environ`.

```python
import os

print(os.environ)
```

`os.environ` behaves like a mapping whose keys and values represent the process environment.

For example:

```python
import os

region = os.environ["AWS_REGION"]
```

This raises `KeyError` if the variable is not present.

That can be useful when a variable is required and the program should fail rather than silently continue with an incomplete configuration.

---

## 4. Reading optional environment variables

Use `os.getenv()` when a variable may be absent and a default is appropriate.

```python
import os

log_level = os.getenv("LOG_LEVEL", "INFO")
```

If the variable exists, its value is returned. Otherwise the supplied default is returned.

Another example:

```python
endpoint = os.getenv("SERVICE_ENDPOINT")

if endpoint is None:
    print("SERVICE_ENDPOINT was not configured")
```

### Design decision

Whether an environment variable should be required or optional is an application configuration decision. Python does not decide that for you.

---

## 5. Required vs optional configuration

A useful DevOps distinction is:

```text
Required setting
    ↓
missing → fail clearly

Optional setting
    ↓
missing → use documented default
```

For example:

```python
import os

api_token = os.environ["API_TOKEN"]
region = os.getenv("REGION", "default-region")
```

The code communicates different expectations for the two settings.

Do not invent a default for a value where silently choosing one could change the meaning of the operation.

---

## 6. Environment variables are strings

Environment variables are represented as strings through `os.environ`.

Therefore this:

```python
import os

workers = os.getenv("WORKERS", "4")
```

produces a string value, not an integer.

If the application expects an integer, convert and validate it at the configuration boundary:

```python
import os

workers = int(os.getenv("WORKERS", "4"))
```

For production automation, add explicit validation rather than assuming conversion will always succeed.

```python
import os

raw_workers = os.getenv("WORKERS", "4")

try:
    workers = int(raw_workers)
except ValueError as exc:
    raise ValueError("WORKERS must be an integer") from exc

if workers < 1:
    raise ValueError("WORKERS must be at least 1")
```

This follows the repository's broader pattern:

```text
read → parse → validate → use
```

---

## 7. Modifying the environment

`os.environ` can also be modified.

```python
import os

os.environ["MODE"] = "test"
```

Deleting an entry unsets the variable from the process environment mapping:

```python
del os.environ["MODE"]
```

Or:

```python
os.environ.pop("MODE", None)
```

Python documents that assigning to `os.environ` automatically updates the process environment through the corresponding operating-system mechanism.

Prefer modifying `os.environ` rather than calling `os.putenv()` directly when working with the mapping.

---

## 8. Environment inheritance and subprocesses

Environment variables are especially important in automation because child processes can receive environment configuration from the parent process.

For example, a program can prepare its environment before launching another process:

```python
import os

os.environ["APP_MODE"] = "production"
```

The exact subprocess behavior and explicit environment passing belong to the later **subprocess** topic.

The key concept here is that environment configuration is part of process execution context.

---

## 9. Current working directory

`os.getcwd()` returns the current working directory as a string.

```python
import os

print(os.getcwd())
```

You can change it with `os.chdir()`:

```python
import os

os.chdir("/tmp")
```

Changing the current working directory changes how relative paths are interpreted by the process.

### DevOps caution

Changing global process state can make code harder to reason about. Prefer explicit `Path` values when a function can operate on a known path instead of depending on the current working directory.

---

## 10. `os` and `pathlib`

The `pathlib` topic introduced `Path` as the preferred high-level abstraction for filesystem paths.

`os` still matters because many operating-system APIs accept path-like objects and because `os` exposes environment and process interfaces.

For example:

```python
from pathlib import Path
import os

config = Path(os.getcwd()) / "config.yaml"
```

You can also pass a `Path` object directly to APIs that accept path-like objects.

```python
from pathlib import Path

with open(Path("config.yaml"), encoding="utf-8") as file:
    content = file.read()
```

Do not convert every `Path` to a string just because an API is operating-system related.

---

## 11. `os.fspath()` and path-like objects

`os.fspath()` returns the filesystem representation of a path-like object.

```python
import os
from pathlib import Path

path = Path("config.yaml")
print(os.fspath(path))
```

This is useful when implementing boundaries that specifically need the filesystem representation.

Most application code should simply pass `Path` objects directly when the receiving API accepts path-like objects.

---

## 12. Listing directory contents

`os.listdir()` returns the names of entries in a directory.

```python
import os

for name in os.listdir("logs"):
    print(name)
```

The returned list is in arbitrary order.

For richer directory-entry information, Python provides `os.scandir()`.

```python
import os

with os.scandir("logs") as entries:
    for entry in entries:
        if entry.is_file():
            print(entry.name)
```

For most new high-level code, `pathlib` is often clearer for path-oriented operations. Learn `os.scandir()` when you specifically need its directory-entry behavior or performance characteristics.

---

## 13. Walking a directory tree

`os.walk()` generates information for each directory in a directory tree.

```python
import os

for root, directories, files in os.walk("project"):
    print(root)
    print(directories)
    print(files)
```

Conceptually:

```text
root directory
     ↓
current directory
     ↓
subdirectories + files
     ↓
next directory
     ↓
...
```

`os.walk()` does not follow symbolic links to directories by default.

If `followlinks=True` is used, the documentation warns that infinite recursion can occur when links create cycles.

---

## 14. Filesystem operations and errors

Many `os` filesystem functions can raise `OSError` or one of its subclasses when an operation cannot be completed.

Example:

```python
import os

try:
    os.chdir("missing-directory")
except OSError as exc:
    print(f"Could not change directory: {exc}")
```

Catch exceptions at a useful boundary and preserve the original error when translating it into an application-specific exception.

Avoid this:

```python
try:
    os.chdir(path)
except Exception:
    pass
```

It hides failures that an automation system may need to report.

---

## 15. `os.stat()` for file metadata

`os.stat()` returns status information for a file or other filesystem path.

```python
import os

info = os.stat("report.txt")
print(info.st_size)
print(info.st_mtime)
```

Useful attributes include values such as:

- `st_size` — file size in bytes for a regular file
- `st_mtime` — most recent content modification time
- `st_atime` — most recent access time
- `st_mode` — file type and mode information

The exact meaning and resolution of timestamps can depend on the operating system and filesystem.

For simple path inspection, `Path.stat()` can be a cleaner interface when you are already using `pathlib`.

---

## 16. Do not use `os.access()` as a pre-check for authorization

The Python documentation specifically warns against using `os.access()` to check whether an operation will succeed and then performing that operation afterward.

This creates a time-of-check/time-of-use race because filesystem state can change between the check and the operation.

Prefer attempting the operation and handling the resulting exception.

For example, prefer the general pattern:

```python
try:
    with open(path, encoding="utf-8") as file:
        content = file.read()
except PermissionError as exc:
    raise RuntimeError(f"Cannot read {path}") from exc
```

rather than treating `os.access()` as a guarantee that the subsequent operation must succeed.

---

## 17. Environment variables and secrets

Environment variables are commonly used as a configuration boundary for automation, but reading a secret from an environment variable does not make the secret inherently safe.

Avoid:

```python
import os

print(os.environ["API_TOKEN"])
```

Do not include secrets in logs, reports, exception messages, screenshots, or committed source files.

Prefer:

```python
import os

api_token = os.environ["API_TOKEN"]
```

and pass the value only to the component that actually needs it.

**Repository design rule:** this repository demonstrates environment-variable access without using real credentials or secret values.

---

## 18. Configuration boundary pattern

A clean DevOps script can isolate environment parsing from the rest of its work.

```python
import os


def load_config() -> dict[str, str]:
    return {
        "environment": os.getenv("APP_ENV", "development"),
        "endpoint": os.environ["SERVICE_ENDPOINT"],
    }


config = load_config()
```

A larger application can then validate this configuration before performing any external action.

```text
process environment
       ↓
load configuration
       ↓
parse values
       ↓
validate
       ↓
application logic
```

This keeps environment-specific concerns out of the core operation.

---

## 19. Environment variables are not automatically typed configuration

This is a common DevOps mistake:

```python
if os.getenv("DEBUG"):
    enable_debugging()
```

A value such as `"false"` is a non-empty string and therefore truthy in Python.

If you need boolean configuration, define and validate an explicit representation.

```python
import os

raw_debug = os.getenv("DEBUG", "false").strip().lower()

if raw_debug == "true":
    debug = True
elif raw_debug == "false":
    debug = False
else:
    raise ValueError("DEBUG must be true or false")
```

The exact accepted values are a repository/application design decision. The important lesson is to avoid relying on Python truthiness for string-based configuration.

---

## 20. Platform information

`os.name` identifies the operating-system-dependent module in use.

```python
import os

print(os.name)
```

For more detailed platform identification, the official documentation points to `sys.platform` and the `platform` module.

Do not assume that one coarse platform identifier is enough for every portability decision.

---

## 21. DevOps pattern: environment-driven behavior

```python
import os

mode = os.getenv("DEPLOYMENT_MODE", "dry-run")

if mode == "dry-run":
    print("No changes will be applied")
elif mode == "apply":
    print("Changes may be applied")
else:
    raise ValueError(f"Unsupported DEPLOYMENT_MODE: {mode}")
```

The important pattern is:

```text
external configuration
       ↓
parse
       ↓
validate allowed values
       ↓
choose behavior
```

Do not allow an arbitrary environment variable to silently control a destructive operation.

---

## 22. DevOps pattern: environment plus filesystem

A script can combine environment configuration with `pathlib`:

```python
import os
from pathlib import Path

workspace = Path(os.getenv("WORKSPACE", "workspace"))
report = workspace / "reports" / "status.txt"

report.parent.mkdir(parents=True, exist_ok=True)
report.write_text("healthy\n", encoding="utf-8")
```

This demonstrates the separation of responsibilities:

- `os` reads process configuration.
- `Path` models filesystem paths.
- filesystem operations perform the actual work.

---

## 23. DevOps pattern: inspect an environment safely

For diagnostics, do not blindly print the entire environment because it may contain credentials or other sensitive values.

Instead, select non-sensitive keys deliberately:

```python
import os

for key in ("APP_ENV", "LOG_LEVEL", "REGION"):
    value = os.getenv(key)
    print(f"{key}={value!r}")
```

For sensitive settings, report presence rather than the value:

```python
import os

print("API_TOKEN configured:", "API_TOKEN" in os.environ)
```

This is a repository design practice for safer diagnostics, not a guarantee that environment variables are secret from every observer of the process.

---

## 24. Common mistakes

### Mistake 1: Treating environment variables as typed values

Environment variables arrive as strings. Parse and validate them.

### Mistake 2: Using a default for a required setting

A made-up default can hide a deployment configuration error.

### Mistake 3: Treating `"false"` as `False`

Non-empty strings are truthy. Parse boolean configuration explicitly.

### Mistake 4: Logging secrets

Never print tokens, passwords, private keys, or other sensitive configuration values.

### Mistake 5: Changing the current working directory unnecessarily

`os.chdir()` changes process-wide state. Prefer explicit paths where practical.

### Mistake 6: Assuming a filesystem check guarantees a later operation

Filesystem state can change between observations and operations.

### Mistake 7: Catching every exception and continuing

This can turn an operational failure into a false success.

### Mistake 8: Using low-level `os` APIs when a higher-level API is clearer

Use `pathlib` for normal path manipulation and `subprocess` for normal external-process execution. Use `os` when its operating-system interface is the appropriate abstraction.

---

## 25. Practical DevOps exercises

### Exercise 1 — Required configuration

Read `SERVICE_ENDPOINT` as a required environment variable and produce a clear error if it is missing.

### Exercise 2 — Integer configuration

Read `WORKERS`, convert it to an integer, and reject values below `1`.

### Exercise 3 — Boolean configuration

Implement explicit parsing for a `DEBUG` environment variable that accepts only `true` and `false`.

### Exercise 4 — Safe diagnostics

Write a diagnostic function that reports whether `API_TOKEN` is configured without printing the token.

### Exercise 5 — Environment-driven workspace

Read `WORKSPACE` from the environment, construct a reports directory with `pathlib`, and create it if necessary.

### Exercise 6 — Directory inspection

Use `os.scandir()` to report regular files in a directory.

### Exercise 7 — Configuration validation boundary

Build a `load_config()` function that reads several environment variables, parses their types, validates them, and returns a configuration object.

---

## 26. Knowledge checks

You should be able to answer these without looking at the file:

1. What is the difference between `os.environ["KEY"]` and `os.getenv("KEY")`?
2. What type do environment-variable values have when read through `os.environ`?
3. Why should environment configuration be parsed and validated at a boundary?
4. Why is `if os.getenv("DEBUG")` unsafe for boolean configuration?
5. What does `os.getcwd()` return?
6. Why can changing the current working directory make code harder to reason about?
7. What is the relationship between `os` and `pathlib`?
8. Why should secrets not be printed while diagnosing environment configuration?
9. Why is `os.access()` not a guarantee that a later filesystem operation will succeed?
10. What does `os.walk()` conceptually provide?

---

## 27. Completion criteria

You are ready to move on when you can:

- read required and optional environment variables
- distinguish missing configuration from explicit empty values when your application requires it
- parse and validate string-based configuration
- modify `os.environ` deliberately
- explain environment inheritance at a high level
- inspect the current working directory
- use `os` alongside `pathlib`
- understand common `OSError` failures
- use `os.scandir()` or `os.walk()` for appropriate filesystem tasks
- avoid leaking secrets in diagnostics
- avoid treating pre-checks as guarantees about later filesystem operations

---

## 28. Verified facts vs repository design

### Verified facts

The official Python documentation establishes that:

- `os.environ` provides access to the process environment.
- `os.getenv()` returns an environment-variable value or a supplied default.
- modifying `os.environ` modifies the process environment through the corresponding operating-system interface.
- `os.getcwd()` returns the current working directory.
- `os.chdir()` changes the current working directory.
- many `os` filesystem functions raise `OSError` or subclasses on failure.
- `os.listdir()` lists directory entries.
- `os.scandir()` provides `DirEntry` objects and can be used as a context manager.
- `os.walk()` traverses directory trees.
- `os.PathLike` provides the filesystem path interface used by path-like objects.
- the documentation warns against using `os.access()` as a pre-check before opening or otherwise operating on a path.

### Repository design decisions

This repository recommends:

- keeping configuration parsing at a clear boundary
- treating required and optional configuration differently
- explicitly parsing boolean and numeric environment variables
- avoiding secret values in diagnostics
- preferring `pathlib` for ordinary high-level path manipulation
- avoiding unnecessary global working-directory changes

These recommendations are engineering guidance for this repository, not claims that Python requires a particular DevOps architecture.

---

## 29. Authoritative references

- Python Standard Library — `os`:  
  https://docs.python.org/3/library/os.html
- Python `pathlib`:  
  https://docs.python.org/3/library/pathlib.html
- Python built-in `open()`:  
  https://docs.python.org/3/library/functions.html#open
- Python exceptions:  
  https://docs.python.org/3/library/exceptions.html

> **Source boundary:** This file focuses on Python's standard-library operating-system interfaces. Container, Kubernetes, CI/CD, cloud-provider, and secret-manager behavior should be documented from their respective authoritative sources when those topics are introduced.