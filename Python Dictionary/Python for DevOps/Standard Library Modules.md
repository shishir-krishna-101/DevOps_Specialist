# Standard Library Modules

Python ships with a large standard library that covers many common automation tasks without adding third-party dependencies. The official library reference groups modules by purpose, including filesystem access, data formats, command-line interfaces, concurrency, networking, runtime services, and more.

> **Source note:** Python module behavior and module availability described here are based on the official Python documentation. The DevOps-oriented grouping and examples are repository design decisions.

---

## 1. The DevOps mental model

A useful way to approach the standard library is by the boundary your automation needs to control:

```text
filesystem       → pathlib / os / shutil / tempfile
configuration    → json / configparser / tomllib
text             → re / textwrap
CLI              → argparse / getpass
processes        → subprocess / signal
logging          → logging
networking       → urllib / socket / ssl
concurrency      → concurrent.futures / threading / queue
runtime          → sys / platform / traceback
collections      → collections / itertools / functools
security helpers → hashlib / hmac / secrets
```

You do not need to memorize the entire standard library. Learn to recognize the problem category and then consult the official module documentation.

---

## 2. Filesystem automation

### `pathlib`

Use `pathlib` for object-oriented filesystem paths.

```python
from pathlib import Path

config = Path("config") / "production.json"

if config.is_file():
    print(config)
```

Use the dedicated `pathlib` topic in this repository for deeper coverage.

Official reference:

https://docs.python.org/3/library/pathlib.html

### `shutil`

`shutil` provides high-level file and directory operations.

```python
import shutil

shutil.copy2("source.txt", "backup.txt")
```

Be explicit about destructive operations such as directory removal.

Official reference:

https://docs.python.org/3/library/shutil.html

### `tempfile`

Use `tempfile` when automation needs temporary files or directories.

```python
from tempfile import TemporaryDirectory

with TemporaryDirectory() as directory:
    print(directory)
```

Temporary resources should have a clear lifetime and cleanup strategy.

Official reference:

https://docs.python.org/3/library/tempfile.html

---

## 3. Operating-system interfaces

### `os`

`os` provides portable interfaces to operating-system functionality.

Common automation areas include environment variables, process-related information, and lower-level filesystem operations.

```python
import os

mode = os.environ.get("APP_MODE", "development")
print(mode)
```

Prefer `pathlib` for path-oriented code when it provides the clearer interface.

Official reference:

https://docs.python.org/3/library/os.html

### `platform`

`platform` exposes identifying information about the underlying platform.

```python
import platform

print(platform.system())
print(platform.machine())
```

Do not build portability assumptions from a single platform property without verifying the behavior you actually need.

Official reference:

https://docs.python.org/3/library/platform.html

---

## 4. Configuration and data formats

### `json`

Use `json` for JSON encoding and decoding.

```python
import json

payload = json.loads('{"enabled": true}')
print(payload["enabled"])
```

Official reference:

https://docs.python.org/3/library/json.html

### `configparser`

`configparser` provides support for INI-style configuration files.

Official reference:

https://docs.python.org/3/library/configparser.html

### `tomllib`

`tomllib` parses TOML data.

```python
import tomllib

with open("pyproject.toml", "rb") as file:
    config = tomllib.load(file)
```

Check the Python version supported by your project before using a module introduced in a newer Python release.

Official reference:

https://docs.python.org/3/library/tomllib.html

---

## 5. Text processing

### `re`

`re` provides regular-expression operations.

```python
import re

match = re.search(r"status=(\w+)", "status=healthy")
if match:
    print(match.group(1))
```

Use regular expressions when the matching problem genuinely requires pattern matching. For simple prefixes, suffixes, or exact comparisons, ordinary string methods are often clearer.

Official reference:

https://docs.python.org/3/library/re.html

### `textwrap`

`textwrap` provides utilities for formatting and wrapping text. It can be useful when generating human-readable reports or CLI output.

Official reference:

https://docs.python.org/3/library/textwrap.html

---

## 6. Command-line automation

### `argparse`

`argparse` supports command-line options, arguments, and subcommands.

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--environment", required=True)
args = parser.parse_args()

print(args.environment)
```

Use the dedicated `Command-Line Interfaces` topic for deeper coverage.

Official reference:

https://docs.python.org/3/library/argparse.html

### `getpass`

`getpass` provides portable password input without putting the password directly in the command line.

```python
from getpass import getpass

password = getpass("Password: ")
```

For production secret management, follow the secret-management system used by the environment rather than treating interactive password input as a complete secret-management solution.

Official reference:

https://docs.python.org/3/library/getpass.html

---

## 7. Processes and signals

### `subprocess`

Use `subprocess` to spawn and manage child processes. The dedicated `subprocess` topic covers this in detail.

Official reference:

https://docs.python.org/3/library/subprocess.html

### `signal`

`signal` provides mechanisms for using signal handlers where supported by the platform and execution model.

This is useful when automation needs to respond to process signals, but signal behavior is platform-dependent and should be verified for the deployment environment.

Official reference:

https://docs.python.org/3/library/signal.html

---

## 8. Logging and diagnostics

### `logging`

Use `logging` instead of scattered `print()` calls for application and automation diagnostics.

```python
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

logger.info("deployment started")
```

Use the dedicated `Logging` topic for handlers, levels, formatting, and structured application design.

Official reference:

https://docs.python.org/3/library/logging.html

### `traceback`

`traceback` provides utilities for extracting and printing Python stack traces. It can be useful when an automation program needs controlled exception reporting.

Official reference:

https://docs.python.org/3/library/traceback.html

---

## 9. Collections and functional tools

### `collections`

`collections` provides specialized container types.

Examples include `Counter`, `defaultdict`, and `deque`.

```python
from collections import Counter

statuses = ["ok", "failed", "ok", "ok"]
print(Counter(statuses))
```

Official reference:

https://docs.python.org/3/library/collections.html

### `itertools`

`itertools` provides iterator building blocks. Use it for composable, iterator-based processing.

Official reference:

https://docs.python.org/3/library/itertools.html

### `functools`

`functools` provides higher-order functions and operations on callable objects.

A common example is `partial()`:

```python
from functools import partial

with_environment = partial(deploy, environment="production")
```

Use functional tools when they make the code clearer; do not introduce them merely to make simple code look more sophisticated.

Official reference:

https://docs.python.org/3/library/functools.html

---

## 10. Dates and time

### `datetime`

`datetime` provides date and time types.

```python
from datetime import datetime, timezone

now = datetime.now(timezone.utc)
print(now)
```

For production systems, be explicit about whether timestamps are timezone-aware and what timezone your interfaces expect.

Official reference:

https://docs.python.org/3/library/datetime.html

### `zoneinfo`

`zoneinfo` provides IANA time-zone support.

Official reference:

https://docs.python.org/3/library/zoneinfo.html

---

## 11. CSV and archives

### `csv`

Use `csv` for CSV reading and writing instead of manually splitting on commas.

```python
import csv

with open("servers.csv", newline="", encoding="utf-8") as file:
    for row in csv.DictReader(file):
        print(row)
```

Official reference:

https://docs.python.org/3/library/csv.html

### `zipfile` and `tarfile`

These modules support ZIP and TAR archive operations.

```python
from zipfile import ZipFile

with ZipFile("artifacts.zip") as archive:
    print(archive.namelist())
```

Treat archive extraction as an input-validation boundary. Do not assume archive contents are safe merely because the archive was successfully opened.

Official references:

https://docs.python.org/3/library/zipfile.html

https://docs.python.org/3/library/tarfile.html

---

## 12. Hashing and secure randomness

### `hashlib`

`hashlib` provides secure hash algorithms and message-digest support.

```python
import hashlib

checksum = hashlib.sha256(b"example").hexdigest()
print(checksum)
```

A cryptographic hash is not automatically an encryption mechanism.

Official reference:

https://docs.python.org/3/library/hashlib.html

### `hmac`

`hmac` implements keyed-hash message authentication.

Official reference:

https://docs.python.org/3/library/hmac.html

### `secrets`

`secrets` is intended for generating cryptographically strong random values suitable for managing secrets such as tokens.

Official reference:

https://docs.python.org/3/library/secrets.html

---

## 13. Networking

### `urllib`

The `urllib` package provides modules for working with URLs and related network operations.

Use it when the standard library is sufficient for the HTTP or URL task. For richer HTTP client requirements, evaluate the project's approved dependencies rather than assuming the standard library is always the best abstraction.

Official reference:

https://docs.python.org/3/library/urllib.html

### `socket`

`socket` provides a low-level networking interface.

It is useful when automation needs direct socket-level behavior rather than a higher-level HTTP abstraction.

Official reference:

https://docs.python.org/3/library/socket.html

### `ssl`

`ssl` provides TLS/SSL support for sockets and related networking functionality.

Official reference:

https://docs.python.org/3/library/ssl.html

---

## 14. Concurrency

### `concurrent.futures`

`concurrent.futures` provides interfaces for launching tasks concurrently.

```python
from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(check_host, hosts))
```

Concurrency should be bounded and should respect the capacity and rate limits of the systems being automated.

Official reference:

https://docs.python.org/3/library/concurrent.futures.html

### `queue`

`queue` provides synchronized queue classes suitable for coordinating work between threads.

Official reference:

https://docs.python.org/3/library/queue.html

---

## 15. Runtime information

### `sys`

`sys` provides system-specific parameters and functions.

Common automation uses include reading command-line arguments and process exit behavior.

Official reference:

https://docs.python.org/3/library/sys.html

### `platform`

Use `platform` when you need identifying information about the operating environment.

### `venv`

`venv` supports creation of Python virtual environments.

Official reference:

https://docs.python.org/3/library/venv.html

---

## 16. How to choose a module

Use this decision process:

```text
What boundary am I automating?
        |
        +-- files/paths ------→ pathlib / shutil / tempfile
        |
        +-- environment ------→ os / platform
        |
        +-- config/data -------→ json / csv / configparser / tomllib
        |
        +-- CLI --------------→ argparse / getpass
        |
        +-- process ----------→ subprocess / signal
        |
        +-- diagnostics ------→ logging / traceback
        |
        +-- time -------------→ datetime / zoneinfo
        |
        +-- network ----------→ urllib / socket / ssl
        |
        +-- concurrency ------→ concurrent.futures / queue
        |
        +-- collections ------→ collections / itertools / functools
        |
        +-- security helpers -→ hashlib / hmac / secrets
```

Then read the official module documentation before relying on an unfamiliar API.

---

## 17. Common mistakes

### Mistake 1: importing a third-party package unnecessarily

First check whether the standard library already provides an adequate solution.

### Mistake 2: using a low-level module when a higher-level one is clearer

For example, use `pathlib` for path manipulation instead of manually constructing path strings.

### Mistake 3: assuming every module behaves identically on every platform

The standard library provides many portability abstractions, but some operating-system functionality is platform-specific.

### Mistake 4: ignoring Python-version requirements

A module or API may be introduced or changed in a particular Python release. Verify the supported Python versions before using it.

### Mistake 5: treating security helpers as complete security architecture

`hashlib`, `hmac`, and `secrets` solve specific primitives. They do not replace a complete authentication, authorization, key-management, or secret-management design.

### Mistake 6: using concurrency without controlling external load

More workers can increase pressure on APIs, hosts, databases, or other infrastructure. Concurrency limits should be an explicit design decision.

---

## 18. Practical exercise: build a standard-library toolkit

Create a small CLI tool that accepts a directory and produces a report containing:

1. discovered files using `pathlib`;
2. file counts grouped by suffix using `collections.Counter`;
3. total bytes using filesystem metadata;
4. a timestamp using `datetime`;
5. structured logging using `logging`;
6. command-line arguments using `argparse`.

Keep the implementation within the Python standard library.

### Extension
Add a temporary output file using `tempfile`, then package the report into an archive using an appropriate standard-library archive module.

---

## 19. Knowledge checks

You should be able to answer:

1. When would you choose `pathlib` over `os.path`?
2. What problem does `shutil` solve?
3. When is `tempfile` preferable to manually creating a temporary path?
4. Which standard-library modules handle JSON, CSV, and TOML parsing?
5. What is the role of `argparse`?
6. When should `logging` be preferred over `print()`?
7. What is the difference between `hashlib`, `hmac`, and `secrets` at a high level?
8. What does `concurrent.futures` provide?
9. Why must platform and Python-version assumptions be verified?
10. Why is knowing the standard library useful for DevOps automation?

---

## 20. Completion criteria

You have completed this topic when you can:

- identify the standard-library module category relevant to an automation problem;
- choose an appropriate high-level module before reaching for a third-party dependency;
- find the authoritative documentation for an unfamiliar module;
- recognize platform and Python-version constraints;
- combine standard-library modules into a small automation tool;
- explain why a module solves a specific problem rather than merely memorizing its API.

---

## 21. Verified facts vs repository design

### Verified facts

- Python distributes a standard library containing modules for filesystem access, data formats, CLI interfaces, concurrency, networking, runtime services, and other common programming tasks.
- `pathlib`, `shutil`, `tempfile`, `os`, `json`, `csv`, `configparser`, `argparse`, `subprocess`, `logging`, `datetime`, `collections`, `itertools`, `functools`, `hashlib`, `hmac`, `secrets`, `urllib`, `socket`, `ssl`, `concurrent.futures`, `queue`, `sys`, and `venv` are documented Python standard-library modules.

### Repository design decisions

- The module groupings in this guide are organized around common DevOps automation boundaries.
- The examples intentionally use generic infrastructure concepts rather than assuming a particular cloud, CI system, operating system, or vendor.
- The repository recommends checking official documentation before relying on unfamiliar APIs.

---

## 22. Authoritative references

- Python Standard Library: https://docs.python.org/3/library/index.html
- File and Directory Access: https://docs.python.org/3/library/filesys.html
- Operating System Interfaces: https://docs.python.org/3/library/os.html
- Command-Line Interface Libraries: https://docs.python.org/3/library/cmdlinelibs.html
- Functional Programming Modules: https://docs.python.org/3/library/functional.html
- `pathlib`: https://docs.python.org/3/library/pathlib.html
- `shutil`: https://docs.python.org/3/library/shutil.html
- `tempfile`: https://docs.python.org/3/library/tempfile.html
- `json`: https://docs.python.org/3/library/json.html
- `csv`: https://docs.python.org/3/library/csv.html
- `argparse`: https://docs.python.org/3/library/argparse.html
- `logging`: https://docs.python.org/3/library/logging.html
- `datetime`: https://docs.python.org/3/library/datetime.html
- `collections`: https://docs.python.org/3/library/collections.html
- `itertools`: https://docs.python.org/3/library/itertools.html
- `functools`: https://docs.python.org/3/library/functools.html
- `hashlib`: https://docs.python.org/3/library/hashlib.html
- `hmac`: https://docs.python.org/3/library/hmac.html
- `secrets`: https://docs.python.org/3/library/secrets.html
- `urllib`: https://docs.python.org/3/library/urllib.html
- `socket`: https://docs.python.org/3/library/socket.html
- `ssl`: https://docs.python.org/3/library/ssl.html
- `concurrent.futures`: https://docs.python.org/3/library/concurrent.futures.html
- `queue`: https://docs.python.org/3/library/queue.html
- `sys`: https://docs.python.org/3/library/sys.html
- `venv`: https://docs.python.org/3/library/venv.html
