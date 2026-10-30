# pathlib

`pathlib` provides an object-oriented way to work with filesystem paths. For DevOps automation, it is useful when scripts need to construct paths, inspect files and directories, traverse trees, and combine path handling with file I/O.

> **Source note:** This topic is based on the official Python `pathlib` documentation. The examples are teaching demonstrations created for this repository.

---

## 1. The mental model

Separate these two concerns:

```text
Path object
   ↓
represents / manipulates a filesystem path
   ↓
filesystem operation
   ↓
read, write, inspect, create, remove, traverse
```

A `Path` object represents a concrete filesystem path for the platform where the code is running. `PurePath` and its platform-specific variants can manipulate paths without performing filesystem I/O.

For most normal application and DevOps code, start with:

```python
from pathlib import Path
```

and use `Path` unless you have a specific reason to use a pure path class.

---

## 2. Creating paths

```python
from pathlib import Path

config = Path("config")
file = Path("config") / "service.conf"

print(config)
print(file)
```

The `/` operator combines path components.

```python
base = Path("/srv/app")
log_file = base / "logs" / "application.log"
```

This is preferable to manually assembling path strings with platform-specific separators.

### Important behavior

If an absolute path is used as a later component, earlier components can be discarded according to the path joining rules.

```python
from pathlib import PurePath

print(PurePath("/srv/app") / "/tmp/data")
```

Do not assume that every component is simply appended.

---

## 3. Relative and absolute paths

A path can be relative:

```python
path = Path("reports/status.txt")
```

or absolute:

```python
path = Path("/var/tmp/status.txt")
```

Check whether a path is absolute:

```python
if path.is_absolute():
    print("absolute path")
```

A relative path is interpreted in relation to the process's current working directory when a filesystem operation uses it.

**DevOps habit:** make the intended path base explicit instead of relying on whatever directory happened to launch the script.

---

## 4. Inspecting path components

`pathlib` exposes useful properties for understanding a path.

```python
from pathlib import Path

path = Path("/srv/app/releases/service.tar.gz")

print(path.name)
print(path.stem)
print(path.suffix)
print(path.parent)
```

Conceptually:

| Property | Purpose |
|---|---|
| `name` | Final path component |
| `parent` | Logical parent path |
| `stem` | Final component without its final suffix |
| `suffix` | Final suffix |
| `suffixes` | List of suffixes |
| `parts` | Path components |

For example, a path such as `service.tar.gz` has a final suffix of `.gz`, while `suffixes` exposes both `.tar` and `.gz`.

---

## 5. Current directory

Use `Path.cwd()` to obtain the current working directory as a `Path` object.

```python
from pathlib import Path

working_directory = Path.cwd()
print(working_directory)
```

This is useful when an automation script intentionally works relative to the directory from which it was launched.

Do not confuse the current working directory with the directory containing the Python source file. They can be different.

---

## 6. Expanding a user path

`Path.home()` returns the current user's home directory as a `Path`.

```python
from pathlib import Path

home = Path.home()
print(home)
```

This can be useful for user-specific automation, but production automation should not assume that a human user's home directory is the correct location for application state.

---

## 7. Joining paths safely

Use path composition instead of string concatenation.

Avoid:

```python
base = "/srv/app"
filename = "config.yaml"
path = base + "/" + filename
```

Prefer:

```python
from pathlib import Path

base = Path("/srv/app")
path = base / "config.yaml"
```

The `Path` approach expresses that the values are filesystem paths rather than arbitrary strings.

---

## 8. Checking whether something exists

```python
from pathlib import Path

path = Path("config.yaml")

if path.exists():
    print("path exists")
```

You can also distinguish common filesystem object types:

```python
if path.is_file():
    print("regular file")

if path.is_dir():
    print("directory")
```

These checks are useful, but remember that filesystem state can change between a check and a later operation.

**Repository design rule:** do not treat `exists()` followed by an operation as an atomic guarantee about the filesystem.

---

## 9. Creating directories

`Path.mkdir()` creates a directory.

```python
from pathlib import Path

Path("reports").mkdir()
```

For nested directories, `parents=True` can create missing parent directories:

```python
Path("output/reports/daily").mkdir(parents=True)
```

If the directory may already exist and that is acceptable, `exist_ok=True` changes the behavior accordingly:

```python
Path("output/reports").mkdir(parents=True, exist_ok=True)
```

Use these options deliberately. They describe the expected state of the directory.

---

## 10. Reading and writing files through `Path`

A `Path` can open a file directly:

```python
from pathlib import Path

path = Path("status.txt")

with path.open("r", encoding="utf-8") as file:
    content = file.read()
```

For simple text operations, `Path` also provides convenience methods:

```python
path.write_text("healthy\n", encoding="utf-8")
```

and:

```python
content = path.read_text(encoding="utf-8")
```

For binary data:

```python
path.write_bytes(b"example")
data = path.read_bytes()
```

These methods are convenient when the operation is simple and the complete content is appropriate for the application.

For more detailed stream control, use `Path.open()` and the normal file-object APIs covered in **Files**.

---

## 11. Iterating through a directory

`Path.iterdir()` yields the directory's immediate contents.

```python
from pathlib import Path

for entry in Path(".").iterdir():
    print(entry)
```

You can inspect each entry:

```python
for entry in Path("logs").iterdir():
    if entry.is_file():
        print(entry)
```

`iterdir()` does not recursively walk every descendant directory. It operates on the immediate contents of the directory.

---

## 12. Finding files with `glob()`

`Path.glob()` matches paths using a glob pattern.

```python
from pathlib import Path

for path in Path("logs").glob("*.log"):
    print(path)
```

For recursive matching, use a pattern such as:

```python
for path in Path("logs").glob("**/*.log"):
    print(path)
```

The pattern describes what you want to find; it does not mean that every filesystem object will necessarily be a regular file.

Check the result when your automation requires a particular object type.

---

## 13. `rglob()` for recursive matching

`Path.rglob()` provides recursive globbing.

```python
from pathlib import Path

for path in Path("project").rglob("*.yaml"):
    print(path)
```

This is convenient for repository and configuration inspection tasks.

Example automation pattern:

```text
repository
   ↓
find matching configuration files
   ↓
inspect each file
   ↓
validate
   ↓
report problems
```

Keep discovery separate from validation so that the code remains easy to test.

---

## 14. Path normalization and `resolve()`

`Path.resolve()` can produce an absolute path and resolve symbolic links according to the platform's filesystem behavior.

```python
from pathlib import Path

path = Path("./config/../config/service.yaml")
resolved = path.resolve()

print(resolved)
```

Do not use `resolve()` merely because an absolute string looks nicer. It can perform filesystem-related resolution and has behavior worth understanding when symlinks or missing paths matter.

Also remember that lexical path manipulation and actual filesystem resolution are different concepts.

---

## 15. Renaming and moving paths

`Path.rename()` renames a filesystem path.

```python
from pathlib import Path

source = Path("old-report.txt")
target = Path("new-report.txt")

source.rename(target)
```

For automation, treat renaming as a filesystem operation that can fail. Do not assume that a target is always writable or that the source always exists.

Handle the relevant `OSError` failures at the boundary where you can provide useful context.

---

## 16. Removing files and directories

A file can be removed with `unlink()`:

```python
from pathlib import Path

Path("temporary.txt").unlink()
```

An empty directory can be removed with `rmdir()`:

```python
Path("empty-directory").rmdir()
```

These operations are destructive.

**DevOps safety rule:** never turn a user-controlled path into a destructive filesystem operation without carefully validating the intended target and the operation's scope.

Do not invent a recursive-delete workflow here. Recursive deletion has additional safety considerations and should be handled explicitly rather than treated as a simple equivalent of `rmdir()`.

---

## 17. Path objects and `open()`

`Path` objects implement the `os.PathLike` interface, so they can be passed to APIs that accept path-like objects.

For example:

```python
from pathlib import Path

path = Path("config.txt")

with open(path, "r", encoding="utf-8") as file:
    content = file.read()
```

This makes `pathlib` compatible with the standard file APIs.

You do not need to convert every `Path` into a string first.

---

## 18. `Path` vs `PurePath`

Most automation code should use `Path` because it represents a path tied to the platform on which the code runs and provides filesystem operations.

Pure path classes are useful when you specifically need path manipulation without filesystem access or need to represent a different path flavour.

For example:

```python
from pathlib import PureWindowsPath

windows_path = PureWindowsPath("C:/Users/example/project")
print(windows_path)
```

A pure path does not access the filesystem.

This distinction is useful in testing and in tooling that needs to reason about paths without touching the machine.

---

## 19. Cross-platform path handling

A major benefit of `pathlib` is that path operations are expressed using path semantics rather than manually embedded operating-system separators.

Prefer:

```python
from pathlib import Path

config = Path("config") / "production" / "service.yaml"
```

instead of:

```python
config = "config/production/service.yaml"
```

The latter may work in many environments, but the former communicates the intended abstraction directly.

### Important limitation

`pathlib` does not make an operating-system-specific filesystem layout portable by itself. Your automation still needs to understand assumptions about permissions, available directories, executable locations, mount points, and other environment-specific behavior.

---

## 20. DevOps pattern: build an application workspace

```python
from pathlib import Path

workspace = Path("workspace")

for directory in (
    workspace / "config",
    workspace / "reports",
    workspace / "artifacts",
):
    directory.mkdir(parents=True, exist_ok=True)
```

The important idea is to represent paths as data and construct child paths from a known base.

---

## 21. DevOps pattern: discover configuration files

```python
from pathlib import Path

config_root = Path("config")

for path in config_root.rglob("*.yaml"):
    if path.is_file():
        print(path)
```

This separates discovery from later processing:

```text
Path root
   ↓
rglob()
   ↓
matching paths
   ↓
file validation
   ↓
read / parse
```

The exact YAML parsing strategy belongs to the later configuration topic.

---

## 22. DevOps pattern: generate a report next to an input

```python
from pathlib import Path

input_file = Path("data/input.txt")
report_file = input_file.with_name("report.txt")

report_file.write_text("processing complete\n", encoding="utf-8")
```

This illustrates a useful design pattern: derive related paths from a `Path` object rather than manually concatenating strings.

---

## 23. Path properties are not validation

Suppose you have:

```python
path = Path("service.yaml")
```

Knowing that:

```python
path.suffix == ".yaml"
```

does **not** prove that the file contains valid YAML.

Likewise:

```python
path.exists()
```

does not prove that the file is readable, correctly formatted, or appropriate for your application.

A robust automation pipeline keeps these concerns separate:

```text
path construction
      ↓
filesystem checks
      ↓
read bytes/text
      ↓
parse
      ↓
validate
      ↓
act
```

---

## 24. Common mistakes

### Mistake 1: Building paths with string concatenation

Fragile:

```python
path = base + "/" + filename
```

Prefer:

```python
path = Path(base) / filename
```

### Mistake 2: Assuming `exists()` guarantees the next operation

Filesystem state can change after the check.

Treat checks as observations, not permanent guarantees.

### Mistake 3: Confusing `Path` with file contents

A `Path` identifies a filesystem location. It does not automatically parse the data stored there.

### Mistake 4: Using `resolve()` without understanding its purpose

Path normalization, absolute paths, and filesystem resolution are related but not identical concepts.

### Mistake 5: Ignoring destructive operations

`unlink()`, `rmdir()`, renaming, and writing can modify system state. Validate targets and handle failures deliberately.

### Mistake 6: Assuming extensions define formats

`.yaml`, `.json`, `.log`, and `.conf` are naming conventions. The extension alone does not validate the content.

---

## 25. Practical DevOps exercises

### Exercise 1 — Workspace builder

Write a script that creates this structure:

```text
workspace/
├── config/
├── reports/
└── artifacts/
```

Use `Path` and `mkdir()`.

### Exercise 2 — Configuration discovery

Search a `config/` directory recursively for `.yaml` files and print their paths.

### Exercise 3 — File classification

Given a directory, report whether each immediate entry is a file or directory.

### Exercise 4 — Report generation

Create a report file beside an input file using a `Path` operation rather than manual string concatenation.

### Exercise 5 — Safe validation boundary

Build a function that receives a path, checks that it exists and is a file, reads it, and converts filesystem failures into an application-level exception while preserving the original cause.

### Exercise 6 — Pure path exercise

Use `PureWindowsPath` to manipulate a Windows-style path without accessing the filesystem.

---

## 26. Knowledge checks

You should be able to answer these without looking at the file:

1. Why is `Path("a") / "b"` preferable to manually joining strings?
2. What is the difference between `Path` and `PurePath`?
3. What does `Path.iterdir()` return conceptually?
4. How does `glob()` differ from `rglob()`?
5. Why does `exists()` not provide an atomic guarantee?
6. What is the difference between `name`, `stem`, and `suffix`?
7. When would `Path.open()` be preferable to `read_text()`?
8. Why should destructive filesystem operations be treated carefully?
9. Does a `.yaml` suffix prove that the file contains valid YAML?
10. Why should path construction be separated from parsing and validation?

---

## 27. Completion criteria

You are ready to move on when you can:

- create and compose paths with `Path`
- distinguish relative and absolute paths
- inspect `name`, `parent`, `stem`, `suffix`, and `parts`
- use `exists()`, `is_file()`, and `is_dir()` appropriately
- create directories with `mkdir()`
- read and write simple text and binary files through `Path`
- iterate through directories with `iterdir()`
- discover files with `glob()` and `rglob()`
- understand the difference between `Path` and pure path classes
- handle filesystem failures deliberately
- separate path handling from parsing and validation

---

## 28. Verified facts vs repository design

### Verified facts

The official Python documentation establishes that:

- `pathlib` provides object-oriented filesystem path classes.
- `Path` is the usual concrete path class for the current platform.
- Pure paths support path manipulation without filesystem I/O.
- The `/` operator can combine path components.
- `Path` provides filesystem-oriented operations such as `exists()`, `is_file()`, `is_dir()`, `iterdir()`, `glob()`, and file-opening helpers.
- Path objects implement the `os.PathLike` interface.

### Repository design decisions

This repository recommends:

- using `Path` as the default path abstraction in normal DevOps scripts
- keeping path construction separate from parsing and validation
- treating filesystem state checks as observations rather than atomic guarantees
- using explicit safety boundaries around destructive filesystem operations

These are engineering recommendations for this learning repository, not additional claims about Python's official behavior.

---

## 29. Authoritative references

- Python `pathlib` documentation: https://docs.python.org/3/library/pathlib.html
- Python file I/O tutorial: https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files
- Python `io` documentation: https://docs.python.org/3/library/io.html

> **Source policy:** When behavior matters, prefer the official Python documentation above over secondary tutorials or copied examples.
