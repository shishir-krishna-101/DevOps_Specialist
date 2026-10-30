# Files

File handling is one of the most practical Python skills for DevOps work. Automation scripts frequently read configuration, inspect logs, generate reports, update state files, process exported data, or write command results to disk.

The goal is not to memorize every file API. Learn the lifecycle:

```text
choose the file mode
        ↓
open the stream
        ↓
read / write / iterate
        ↓
handle failures
        ↓
close the resource
```

> **Source note:** This topic is based on the official Python documentation for `open()`, file objects, and the `io` module. Examples in this file are teaching demonstrations created for this repository; they are not copied from an external repository.

---

## 1. The mental model

A file on disk is persistent data. Python interacts with it through a **file object**, which is a kind of stream.

For ordinary file automation, think in terms of two questions:

1. **What kind of data am I handling?**
   - Text → `str`
   - Binary → `bytes`
2. **What do I want to do with it?**
   - Read
   - Write
   - Append
   - Read and write

Python's `io` documentation describes three broad I/O categories: text I/O, binary I/O, and raw I/O. Most application and DevOps scripts should work at the higher-level text or binary interfaces rather than manipulating raw I/O directly.

---

## 2. Opening a file

The built-in `open()` function returns a file object.

Basic form:

```python
file = open(filename, mode)
```

A more explicit text-file form is:

```python
file = open("config.txt", "r", encoding="utf-8")
```

### Common modes

| Mode | Meaning |
|---|---|
| `r` | Read text. The default mode. |
| `w` | Write text. An existing file is truncated. |
| `a` | Append text to the end of the file. |
| `r+` | Read and write without the `w` mode's truncation behavior. |
| `b` | Binary mode modifier, such as `rb` or `wb`. |

Mode characters can be combined according to the `open()` interface. For example:

```python
open("image.jpg", "rb")
open("output.bin", "wb")
```

### Important distinction: `w` vs `a`

This is a common automation mistake.

```python
with open("report.txt", "w", encoding="utf-8") as file:
    file.write("new report\n")
```

`w` is for writing and truncates an existing file.

```python
with open("report.txt", "a", encoding="utf-8") as file:
    file.write("another line\n")
```

`a` writes new data at the end of the file.

**DevOps rule:** choose the mode based on whether replacing existing content is intentional.

---

## 3. Always manage the file lifecycle

Prefer a `with` statement when working with files:

```python
with open("report.txt", "r", encoding="utf-8") as file:
    content = file.read()
```

The file is closed when the `with` block finishes, including when an exception occurs during the block.

This is one of the most important habits in Python automation.

### Why not just call `close()`?

This works:

```python
file = open("report.txt", "r", encoding="utf-8")
try:
    content = file.read()
finally:
    file.close()
```

But the `with` form expresses the resource lifecycle more clearly:

```python
with open("report.txt", "r", encoding="utf-8") as file:
    content = file.read()
```

The `io.IOBase` hierarchy implements the context-manager protocol, which is why file objects work with `with`.

---

## 4. Text files

Text I/O works with Python `str` objects.

```python
with open("app.conf", "r", encoding="utf-8") as file:
    content = file.read()
```

When the underlying data is bytes, text I/O handles encoding and decoding for you.

### Encoding matters

For automation that exchanges text across machines, explicitly choosing an encoding is often clearer than relying on an environment-dependent default.

```python
with open("inventory.txt", "r", encoding="utf-8") as file:
    inventory = file.read()
```

Writing text:

```python
with open("summary.txt", "w", encoding="utf-8") as file:
    file.write("deployment completed\n")
```

The official documentation describes UTF-8 as the default encoding in the documented Python UTF-8 mode context, while also documenting locale-dependent defaults when UTF-8 mode is disabled. For repository examples, explicitly specifying `encoding="utf-8"` makes the intended encoding visible.

---

## 5. Binary files

Binary I/O works with `bytes` rather than `str`.

```python
with open("artifact.bin", "rb") as file:
    data = file.read()
```

Writing binary data:

```python
payload = b"example data"

with open("artifact.bin", "wb") as file:
    file.write(payload)
```

Binary mode does not perform text encoding/decoding or newline translation.

Use binary mode for data that should be treated as bytes rather than decoded text.

Examples include binary artifacts and other non-text data.

### Do not mix text and binary assumptions

A text stream expects `str`:

```python
with open("notes.txt", "w", encoding="utf-8") as file:
    file.write("hello")
```

A binary stream expects bytes-like data:

```python
with open("notes.bin", "wb") as file:
    file.write(b"hello")
```

Passing the wrong data type to a stream can raise `TypeError`.

---

## 6. Reading an entire file

`read()` reads data from the stream.

```python
with open("config.txt", "r", encoding="utf-8") as file:
    content = file.read()
```

This is convenient when the file is small enough for your application to process as one value.

### Read a limited amount

```python
with open("data.txt", "r", encoding="utf-8") as file:
    chunk = file.read(100)
```

The exact behavior depends on the stream and mode, so treat the size as a request for a limited amount rather than as a universal guarantee about application-level records.

### Empty result

At end-of-file, repeated reads can return an empty string for text streams:

```python
with open("empty.txt", "r", encoding="utf-8") as file:
    content = file.read()
```

An empty string is therefore a possible valid result and should not automatically be treated as an error.

---

## 7. Reading one line at a time

Use `readline()` when you want to read a line from the stream.

```python
with open("deployments.txt", "r", encoding="utf-8") as file:
    line = file.readline()
```

You can also iterate directly over the file:

```python
with open("deployments.txt", "r", encoding="utf-8") as file:
    for line in file:
        print(line.rstrip("\n"))
```

For operational data, direct iteration is often a useful pattern because the code can process records incrementally instead of first constructing one large string.

---

## 8. `readlines()` and lists of lines

`readlines()` returns the lines as a list.

```python
with open("hosts.txt", "r", encoding="utf-8") as file:
    lines = file.readlines()
```

You can also build a list from direct iteration:

```python
with open("hosts.txt", "r", encoding="utf-8") as file:
    lines = list(file)
```

Use a list when later logic genuinely needs all records available at once.

If you only need to process each line once, direct iteration is usually the simpler mental model.

---

## 9. Writing files

The `write()` method writes data to a stream.

```python
with open("status.txt", "w", encoding="utf-8") as file:
    file.write("healthy\n")
```

`write()` expects the appropriate data type for the stream.

### Multiple writes

```python
with open("status.txt", "w", encoding="utf-8") as file:
    file.write("service: api\n")
    file.write("status: healthy\n")
```

### `writelines()`

A file object also provides `writelines()` for writing an iterable of strings.

```python
lines = [
    "service: api\n",
    "status: healthy\n",
]

with open("status.txt", "w", encoding="utf-8") as file:
    file.writelines(lines)
```

`writelines()` does not automatically add line separators between strings. Include the separators when the output format requires them.

---

## 10. Newlines

Text I/O can perform newline translation between the platform representation and the text exposed by Python.

For ordinary text processing, you can usually let the text I/O layer handle this behavior.

When newline behavior is part of the file format or interoperability requirement, consult the `open()` documentation for its `newline` parameter instead of assuming that every platform represents line endings identically.

### Practical habit

For line-oriented automation, decide whether your program cares about the newline character itself.

If not, process the logical line rather than depending on a particular platform's physical line-ending representation.

---

## 11. File position: `tell()` and `seek()`

Streams maintain a current position.

`tell()` reports the current position:

```python
with open("data.txt", "r", encoding="utf-8") as file:
    print(file.tell())
    file.read(5)
    print(file.tell())
```

`seek()` changes the stream position:

```python
with open("data.txt", "r", encoding="utf-8") as file:
    file.seek(0)
    content = file.read()
```

The exact meaning of positions depends on the stream and mode. In particular, text streams have encoding and newline behavior that makes treating positions as simple character indexes unsafe in general.

**Learning goal:** understand that a stream has a position and that `seek()`/`tell()` operate on that stream position; use the detailed `io` and `open()` documentation when building non-trivial random-access logic.

---

## 12. Errors are part of file automation

File operations can fail.

Typical causes include:

- the file does not exist
- the process lacks the required permissions
- the path refers to something unexpected
- the storage operation fails
- the requested operation is unsupported by the stream

Python's I/O documentation states that operations that formerly raised `IOError` now raise `OSError`, with `IOError` being an alias of `OSError`.

A basic boundary can therefore be:

```python
try:
    with open("config.txt", "r", encoding="utf-8") as file:
        content = file.read()
except OSError as exc:
    print(f"could not read configuration: {exc}")
```

### Catch narrowly

Do not wrap an entire automation program in one broad exception handler merely to make it appear successful.

Prefer handling the failure where you have enough context to decide what it means.

```python
try:
    with open("inventory.txt", "r", encoding="utf-8") as file:
        inventory = file.read()
except OSError as exc:
    raise RuntimeError("inventory file could not be read") from exc
```

This preserves the original exception as the cause while adding application-level context.

---

## 13. Resource cleanup is not optional

A file object represents an external resource.

Good automation makes the ownership and cleanup boundary obvious:

```python
with open("output.txt", "w", encoding="utf-8") as file:
    file.write("done\n")
```

Avoid keeping file handles open longer than necessary.

Instead of:

```python
file = open("data.txt", "r", encoding="utf-8")
content = file.read()
# lots of unrelated work...
file.close()
```

prefer:

```python
with open("data.txt", "r", encoding="utf-8") as file:
    content = file.read()

# unrelated work happens after the resource is closed
```

This makes the lifetime of the resource visible in the code.

---

## 14. DevOps pattern: process a configuration file

A simple file-reading boundary can be separated from the business logic:

```python
def read_config(path):
    try:
        with open(path, "r", encoding="utf-8") as file:
            return file.read()
    except OSError as exc:
        raise RuntimeError(f"unable to read configuration file: {path}") from exc


config_text = read_config("service.conf")
print(config_text)
```

The important design idea is not the exact function. It is the separation:

```text
filesystem access
        ↓
text content
        ↓
parsing / validation
        ↓
automation decision
```

This makes it easier to test parsing separately from filesystem failures.

---

## 15. DevOps pattern: generate a report

```python
services = [
    ("api", "healthy"),
    ("worker", "healthy"),
    ("scheduler", "degraded"),
]

with open("service-report.txt", "w", encoding="utf-8") as file:
    for name, status in services:
        file.write(f"{name}: {status}\n")
```

The file is deliberately closed as soon as report generation finishes.

For more complex output formats, keep file I/O separate from the logic that determines what the report should contain.

---

## 16. DevOps pattern: process a large line-oriented input

Instead of immediately loading every line into a list:

```python
with open("events.log", "r", encoding="utf-8") as file:
    for line in file:
        if "ERROR" in line:
            print(line.rstrip("\n"))
```

This demonstrates a useful automation principle:

> If you only need to inspect each record once, iterate over the stream rather than materializing all records unnecessarily.

This is a repository design recommendation, not a claim that every large-file workload will fit in memory or that iteration alone solves all scalability concerns.

---

## 17. Appending operational information

Appending can be useful for simple state or report accumulation:

```python
with open("deployment-history.txt", "a", encoding="utf-8") as file:
    file.write("deployment completed\n")
```

Before choosing append mode, ask:

- Is the file supposed to grow indefinitely?
- Is another process also writing it?
- Is there a defined file format?
- Do I need structured logging instead?
- What happens if the operation fails halfway through?

For production logging, use Python's `logging` facilities rather than turning arbitrary files into ad-hoc log files. The dedicated **Logging** topic covers that later.

---

## 18. Files are not automatically configuration formats

A file only gives you bytes/text and I/O operations. The content still needs a format.

For example, this is plain text:

```text
environment=production
region=ap-south-1
```

Python can read it as text, but the application still needs parsing and validation rules.

A useful automation pipeline is:

```text
file
 ↓
text
 ↓
parse
 ↓
validate
 ↓
normalized configuration
```

Do not confuse **reading a file** with **understanding its contents**.

JSON and YAML configuration are covered in later topics.

---

## 19. File I/O and secrets

Avoid treating files as automatically safe places for secrets.

Examples of sensitive data include:

- credentials
- access tokens
- private keys
- generated authentication material
- environment-specific secret configuration

If a script reads sensitive material from a file, think about:

- who can read the file
- where the file is stored
- whether the file is committed to Git
- whether the data should instead come from a secret-management mechanism
- whether the value could accidentally be logged

**Repository design rule:** educational examples in this repository should use placeholder values rather than real credentials.

---

## 20. Common mistakes

### Mistake 1: Using `w` when you meant append

```python
with open("history.txt", "w", encoding="utf-8") as file:
    file.write("new event\n")
```

This can truncate an existing file.

Use `a` when append semantics are actually required.

### Mistake 2: Forgetting `with`

```python
file = open("data.txt", "r", encoding="utf-8")
```

Opening a file is not the end of the operation. The resource must eventually be closed.

Prefer:

```python
with open("data.txt", "r", encoding="utf-8") as file:
    data = file.read()
```

### Mistake 3: Treating binary data as text

```python
with open("artifact.bin", "r", encoding="utf-8") as file:
    data = file.read()
```

This applies text decoding to data that may not be text.

Use the mode appropriate for the data.

### Mistake 4: Ignoring encoding

```python
open("config.txt", "r")
```

This leaves the encoding choice implicit.

For portable application data, make the intended encoding explicit when appropriate:

```python
open("config.txt", "r", encoding="utf-8")
```

### Mistake 5: Catching every exception

```python
try:
    with open("data.txt", "r", encoding="utf-8") as file:
        process(file.read())
except Exception:
    pass
```

This can hide programming errors and operational failures.

Catch the failure you can actually handle.

### Mistake 6: Reading everything by default

```python
content = file.read()
```

`read()` is useful, but it should be a deliberate choice. For line-oriented processing, direct iteration may better match the task.

### Mistake 7: Mixing file I/O with business logic

A function that simultaneously opens files, parses data, validates it, makes infrastructure decisions, and prints results becomes harder to test.

Separate the boundaries where practical.

---

## 21. A practical DevOps example

Suppose a script receives a text file containing service names and needs to produce a report.

```python
def read_services(path):
    services = []

    try:
        with open(path, "r", encoding="utf-8") as file:
            for line in file:
                name = line.strip()
                if name:
                    services.append(name)
    except OSError as exc:
        raise RuntimeError(f"unable to read services from {path}") from exc

    return services


def write_report(path, services):
    with open(path, "w", encoding="utf-8") as file:
        for service in services:
            file.write(f"service={service}\n")


services = read_services("services.txt")
write_report("report.txt", services)
```

The important architecture is:

```text
services.txt
     ↓
read_services()
     ↓
validated/normalized list
     ↓
write_report()
     ↓
report.txt
```

The repository intentionally keeps this example simple. A real automation system would usually add explicit validation, structured output, logging, and tests.

---

## 22. Files vs paths

A **path** identifies where something is located.

A **file object** represents an open I/O stream used to interact with the content.

Conceptually:

```text
path
 ↓
open()
 ↓
file object / stream
 ↓
read or write
```

For example:

```python
path = "reports/status.txt"

with open(path, "r", encoding="utf-8") as file:
    content = file.read()
```

The next topic, **pathlib**, focuses on representing and manipulating filesystem paths. Keep that distinction clear: `pathlib` works primarily with paths; `open()` gives you the stream used for file I/O.

---

## 23. Verified facts vs repository design

### Verified facts

The official Python documentation supports these points:

- `open()` returns a file object.
- `r`, `w`, `a`, and `r+` have distinct read/write behaviors.
- `b` selects binary mode.
- Text streams work with `str`; binary streams work with `bytes`.
- Text I/O handles encoding/decoding and newline translation.
- File objects support the context-manager protocol.
- File streams support operations such as `read()`, `write()`, `readline()`, `seek()`, and `tell()` where supported by the stream.
- I/O failures are represented through the `OSError` hierarchy, with `IOError` as an alias of `OSError`.

### Repository design decisions

This repository recommends that learners:

- prefer `with open(...)` for ordinary file handling
- make the intended text encoding explicit in examples
- separate file access from parsing and business logic
- catch file-related failures at meaningful boundaries
- process line-oriented input incrementally when that matches the task
- avoid putting real secrets into educational files

These are learning and engineering recommendations, not additional Python language guarantees.

---

## 24. Exercises

### Exercise 1 — Read a configuration file

Create a script that:

1. opens a text file using `with`
2. explicitly selects UTF-8
3. reads its contents
4. prints the contents
5. reports a useful error if the file cannot be opened

### Exercise 2 — Count non-empty lines

Read a text file and count only lines containing non-whitespace characters.

Do not load the whole file into a list first.

### Exercise 3 — Generate a report

Given a list of service names, write one service per line to a report file.

Make sure an existing report is intentionally replaced rather than accidentally appended.

### Exercise 4 — Append deployment history

Write a deployment event to a history file using append mode.

Then explain why `w` would be dangerous for this use case.

### Exercise 5 — Binary copy

Read bytes from one file and write them to another file using binary mode.

Explain why text mode is not appropriate for arbitrary binary data.

### Exercise 6 — Separate I/O from parsing

Create two functions:

```text
read_text(path) -> str
parse_services(text) -> list[str]
```

Test the parsing function without opening a real file.

---

## 25. Knowledge check

1. What does `open()` return?
2. What is the difference between `r`, `w`, and `a`?
3. Why can `w` be dangerous when updating an existing file?
4. What is the difference between text and binary I/O?
5. Why should you usually use `with open(...)`?
6. What type of data does a text stream normally work with?
7. What type of data does a binary stream normally work with?
8. What is the role of `encoding` when opening a text file?
9. What does iterating over a file object provide?
10. What do `seek()` and `tell()` represent?
11. Why should file-related exceptions be handled at an appropriate boundary?
12. Why is reading a file different from parsing its contents?
13. When is direct iteration preferable to `readlines()`?
14. Why should secrets not be casually stored in repository files?
15. What is the difference between a filesystem path and an open file object?

---

## 26. Completion criteria

You are ready to move on when you can:

- explain the difference between a path and a file object
- choose an appropriate file mode
- use `with open(...)` confidently
- read and write text files
- work with binary data using binary mode
- explain why encoding matters for text
- iterate over a file line by line
- distinguish replacement from append behavior
- use `seek()` and `tell()` at a basic level
- recognize `OSError` as an important file-I/O failure boundary
- separate file access from parsing and business logic
- explain when loading an entire file is appropriate and when incremental processing is a better fit

---

## 27. Authoritative references

- Python Tutorial — Input and Output:  
  https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files
- Python Built-in Functions — `open()`:  
  https://docs.python.org/3/library/functions.html#open
- Python Standard Library — `io`:  
  https://docs.python.org/3/library/io.html

For path manipulation, continue with the next topic in this track: **pathlib**.
