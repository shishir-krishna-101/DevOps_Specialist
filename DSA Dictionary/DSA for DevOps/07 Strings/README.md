# 07 — Strings

DevOps work often involves reading text:

- log lines;
- command output;
- service names;
- hostnames;
- configuration values;
- job status messages.

Python strings provide operations for searching, splitting, cleaning, and combining text. This lesson introduces the basics and uses simple log-like examples to practise extracting information.

> **Scope note:** The examples use deliberately simple, fictional text. They are not production log parsers, and they do not claim to support every logging format, timestamp format, encoding, or multiline message.

> **Source policy:** Python string behavior described here is based on the official Python documentation linked at the end.

---

## 1. What is a string?

A string is text represented in Python using quotes.

```python
service_name = "nginx"
status = 'running'
message = "Service nginx is running"
```

Single and double quotes can both create strings. Choose the style that makes the text easiest to read.

You can print a string:

```python
print(message)
```

Output:

```text
Service nginx is running
```

Strings are **immutable**. That means you cannot change one character inside an existing string. String methods that appear to modify text return a new string instead.

## 2. String length

Use `len()` to find the number of characters in a string:

```python
service_name = "nginx"

print(len(service_name))
```

Output:

```text
5
```

This can help validate simple inputs, such as checking whether a service name is empty:

```python
service_name = ""

if len(service_name) == 0:
    print("Service name is empty")
```

For this particular check, Python also allows the simpler condition `if not service_name:`.

## 3. Access characters with indexing

Each character has a position called an index. Python string indexes start at zero.

```python
hostname = "web-01"

print(hostname[0])
print(hostname[1])
print(hostname[4])
```

Output:

```text
w
e
0
```

Negative indexes count from the end:

```python
print(hostname[-1])
print(hostname[-2])
```

Output:

```text
1
0
```

### Avoid an index error

An index outside the valid range raises `IndexError`.

```python
hostname = "web-01"

# hostname[20]  # Invalid: there is no character at index 20.
```

Check the length or use a safe operation when the input may be empty or shorter than expected.

## 4. Extract part of a string with slicing

A slice selects a range of characters:

```python
hostname = "web-01"

print(hostname[0:3])
print(hostname[4:6])
```

Output:

```text
web
01
```

The start index is included and the stop index is excluded.

A few more examples:

```python
value = "production"

print(value[:4])   # From the beginning up to index 4
print(value[4:])   # From index 4 to the end
print(value[-3:])  # Last three characters
```

Output:

```text
prod
uction
ion
```

Slicing is useful when a text format has a known, fixed layout. For variable log messages, methods such as `split()` are often easier to understand.

## 5. Search within a string

Use `in` to check whether a smaller string appears inside a larger string:

```python
log_line = "ERROR service=nginx message=connection_failed"

if "ERROR" in log_line:
    print("An error marker was found")
```

Output:

```text
An error marker was found
```

You can also use `not in`:

```python
if "DEBUG" not in log_line:
    print("This line does not contain DEBUG")
```

### Find a position with `find()`

`find()` returns the lowest index where the substring is found. It returns `-1` if it is not found.

```python
message = "service nginx failed"

position = message.find("nginx")

print(position)
print(message.find("postgres"))
```

Output:

```text
8
-1
```

### Difference between `find()` and `index()`

- `find()` returns `-1` when the substring is absent.
- `index()` raises `ValueError` when the substring is absent.

```python
message = "service nginx failed"

print(message.find("redis"))  # -1

# message.index("redis")  # Raises ValueError.
```

For beginner text-processing tasks where absence is normal, `find()` is often convenient.

## 6. Check the beginning and end of text

Use `startswith()` and `endswith()` when the position matters.

```python
log_line = "ERROR service=nginx"

print(log_line.startswith("ERROR"))
print(log_line.endswith("nginx"))
```

Output:

```text
True
True
```

This can help classify simple lines by a known prefix:

```python
if log_line.startswith("ERROR"):
    print("Record this as an error line")
```

This example only checks the text prefix. It does not implement a complete log severity parser.

## 7. Split a line into pieces

A common beginner task is to split a simple line into fields.

Consider this fictional log line:

```text
ERROR nginx connection_failed
```

Use `split()`:

```python
log_line = "ERROR nginx connection_failed"

parts = log_line.split()

print(parts)
```

Output:

```python
["ERROR", "nginx", "connection_failed"]
```

By default, `split()` separates on runs of whitespace.

You can access the pieces by index:

```python
level = parts[0]
service = parts[1]
message = parts[2]

print(level)
print(service)
print(message)
```

Output:

```text
ERROR
nginx
connection_failed
```

### Split using a specific separator

If fields are separated by a comma, provide the separator:

```python
record = "nginx,production,healthy"

fields = record.split(",")

print(fields)
```

Output:

```python
["nginx", "production", "healthy"]
```

The separator must match the actual input format. Splitting arbitrary log text on spaces is not a general-purpose parser: messages may themselves contain spaces or unexpected fields.

## 8. Clean text with `strip()`

Text read from files or entered by users may contain spaces or newline characters at the beginning or end.

Use `strip()` to remove leading and trailing whitespace:

```python
raw_hostname = "  web-01\n"

hostname = raw_hostname.strip()

print(hostname)
```

Output:

```text
web-01
```

Related methods:

- `lstrip()` removes leading whitespace.
- `rstrip()` removes trailing whitespace.
- `strip()` removes whitespace from both ends.

These methods return new strings. They do not modify the original string.

```python
raw_value = "  staging  "

clean_value = raw_value.strip()

print(raw_value)
print(clean_value)
```

Output:

```text
  staging  
staging
```

## 9. Normalize simple text with `lower()` and `upper()`

String comparisons are case-sensitive:

```python
print("ERROR" == "error")
```

Output:

```text
False
```

For a simple case-insensitive comparison, convert both values to the same case:

```python
status = "Error"

if status.lower() == "error":
    print("Error status detected")
```

Output:

```text
Error status detected
```

You can also use `upper()`:

```python
print("warning".upper())
```

Output:

```text
WARNING
```

**Important:** Lowercasing or uppercasing is a normalization choice for a particular field. Do not assume that every identifier, password, token, path, or value should be changed to lowercase. Apply normalization only when the relevant system or data format defines that behavior.

## 10. Replace part of a string

Use `replace(old, new)` to create a new string with matching occurrences replaced:

```python
message = "deployment status: pending"

updated_message = message.replace("pending", "complete")

print(updated_message)
```

Output:

```text
deployment status: complete
```

The original string is unchanged:

```python
print(message)
```

Output:

```text
deployment status: pending
```

You can provide a third argument to limit the number of replacements:

```python
text = "retry retry retry"

print(text.replace("retry", "attempt", 1))
```

Output:

```text
attempt retry retry
```

Be careful when replacing text in configuration or command strings. A broad replacement may change more content than intended.

## 11. Join strings together

Use `join()` to combine an iterable of strings with a separator:

```python
services = ["nginx", "postgres", "redis"]

summary = ", ".join(services)

print(summary)
```

Output:

```text
nginx, postgres, redis
```

The separator is the string before `.join()`.

Another example:

```python
fields = ["ERROR", "nginx", "connection_failed"]

log_line = " ".join(fields)

print(log_line)
```

Output:

```text
ERROR nginx connection_failed
```

All elements passed to `join()` must be strings. Convert non-string values explicitly when necessary:

```python
attempts = 3

message = "attempts=" + str(attempts)

print(message)
```

Output:

```text
attempts=3
```

## 12. Format a useful message with an f-string

An f-string lets you place expressions inside a string:

```python
service = "nginx"
status = "healthy"

message = f"Service {service} is {status}"

print(message)
```

Output:

```text
Service nginx is healthy
```

You can include numeric values:

```python
failed_jobs = 3
total_jobs = 10

message = f"{failed_jobs} of {total_jobs} jobs failed"

print(message)
```

Output:

```text
3 of 10 jobs failed
```

F-strings are useful for readable status messages and diagnostic output.

## 13. Mini project — extract fields from a simple log line

Now combine several string operations.

**Task:** Given a line with exactly three whitespace-separated fields, extract the log level, service name, and message.

Example input:

```text
ERROR nginx connection_failed
```

Expected result:

```python
{
    "level": "ERROR",
    "service": "nginx",
    "message": "connection_failed"
}
```

### Step 1 — Write the function

```python
def parse_simple_log_line(line):
    parts = line.strip().split()

    if len(parts) != 3:
        return None

    return {
        "level": parts[0],
        "service": parts[1],
        "message": parts[2]
    }


log_line = "ERROR nginx connection_failed"

result = parse_simple_log_line(log_line)
print(result)
```

### Step 2 — Understand each line

1. `def parse_simple_log_line(line):` defines a function that accepts one string.
2. `line.strip()` removes leading and trailing whitespace.
3. `split()` separates the text into whitespace-delimited pieces.
4. `len(parts) != 3` checks whether the input has exactly three fields.
5. `return None` indicates that this teaching parser rejected the input shape.
6. The dictionary maps field names to their extracted values.

### Step 3 — Dry run

Input:

```text
ERROR nginx connection_failed
```

| Step | Operation | Result |
|---|---|---|
| 1 | `strip()` | `ERROR nginx connection_failed` |
| 2 | `split()` | `["ERROR", "nginx", "connection_failed"]` |
| 3 | `len(parts)` | `3` |
| 4 | Build dictionary | Three named fields |

### Step 4 — Test valid and invalid input

```python
assert parse_simple_log_line(
    "ERROR nginx connection_failed"
) == {
    "level": "ERROR",
    "service": "nginx",
    "message": "connection_failed"
}

assert parse_simple_log_line(
    " INFO redis started "
) == {
    "level": "INFO",
    "service": "redis",
    "message": "started"
}

assert parse_simple_log_line("ERROR nginx") is None
assert parse_simple_log_line("") is None

print("All checks passed")
```

Expected output:

```text
All checks passed
```

### Limitation of this parser

This is intentionally a tiny exercise. It assumes exactly three whitespace-separated fields and a one-word message. It will not correctly parse a message such as `connection failed unexpectedly`, quoted fields, structured JSON logs, multiline logs, or arbitrary vendor-specific formats.

A real parser must follow the actual log format's documented grammar. Do not use this demonstration as a production log parser.

### Complexity

Let `n` be the number of characters in the input line.

- **Time:** O(n) for trimming and splitting the line, plus constant work to inspect three fields under this exercise's fixed-field assumption.
- **Additional space:** O(n) for the resulting pieces and extracted strings in the general case.

## 14. Common mistakes

| Mistake | Why it is a problem | Better habit |
|---|---|---|
| Trying to change `text[0]` | Strings are immutable | Build a new string |
| Using an invalid index | Raises `IndexError` | Check length or use slicing |
| Assuming `find()` raises an error | It returns `-1` when absent | Check for `-1` |
| Assuming `index()` returns `-1` | It raises `ValueError` when absent | Use `find()` if absence is normal |
| Splitting on a separator that is not present | The result may not have the expected fields | Validate the input format and number of parts |
| Forgetting to trim input | Spaces or newlines can remain | Use `strip()` where appropriate |
| Assuming string comparisons ignore case | Comparisons are case-sensitive | Normalize only when appropriate |
| Assuming `replace()` edits the original string | Strings are immutable | Store the returned string |
| Passing numbers directly to `join()` | `join()` expects strings | Convert values to strings first |
| Treating a simple split as a production parser | Real formats can contain quoting, spaces, and escaping | Follow the authoritative format specification |

## 15. Practice exercises

Try these before opening the suggested solutions.

### Exercise 1 — Count characters

Write a function that returns the length of a service name.

### Exercise 2 — Find an error marker

Write a function that returns `True` if a line contains the text `"ERROR"`.

### Exercise 3 — Extract a hostname prefix

Given `"web-01"`, use slicing to extract `"web"`.

### Exercise 4 — Clean a value

Remove leading and trailing whitespace from `"  production  "`.

### Exercise 5 — Normalize a status

Convert `"Warning"` to lowercase and compare it with `"warning"`.

### Exercise 6 — Split a simple record

Split `"nginx,production,healthy"` using the comma separator.

### Exercise 7 — Build a message

Use an f-string to create `"Service redis is running"` from two variables.

### Exercise 8 — Parse a simple log line

Complete the three-field parser from the mini project and test a line with the wrong number of fields.

---

## Suggested solutions

<details>
<summary>Exercise 1 — Count characters</summary>

```python
def text_length(value):
    return len(value)


assert text_length("nginx") == 5
assert text_length("") == 0
```

</details>

<details>
<summary>Exercise 2 — Find an error marker</summary>

```python
def contains_error(line):
    return "ERROR" in line


assert contains_error("ERROR nginx failed") is True
assert contains_error("INFO nginx started") is False
```

</details>

<details>
<summary>Exercise 3 — Extract a hostname prefix</summary>

```python
hostname = "web-01"

assert hostname[:3] == "web"
```

</details>

<details>
<summary>Exercise 4 — Clean a value</summary>

```python
value = "  production  "

assert value.strip() == "production"
```

</details>

<details>
<summary>Exercise 5 — Normalize a status</summary>

```python
status = "Warning"

assert status.lower() == "warning"
```

</details>

<details>
<summary>Exercise 6 — Split a simple record</summary>

```python
record = "nginx,production,healthy"

fields = record.split(",")

assert fields == ["nginx", "production", "healthy"]
```

</details>

<details>
<summary>Exercise 7 — Build a message</summary>

```python
service = "redis"
status = "running"

message = f"Service {service} is {status}"

assert message == "Service redis is running"
```

</details>

<details>
<summary>Exercise 8 — Parse a simple log line</summary>

```python
def parse_simple_log_line(line):
    parts = line.strip().split()

    if len(parts) != 3:
        return None

    return {
        "level": parts[0],
        "service": parts[1],
        "message": parts[2]
    }


assert parse_simple_log_line(
    "ERROR nginx connection_failed"
) == {
    "level": "ERROR",
    "service": "nginx",
    "message": "connection_failed"
}

assert parse_simple_log_line("ERROR nginx") is None
```

</details>

## 16. Lesson completion checklist

- [ ] Explain what a Python string is.
- [ ] Explain why strings are immutable.
- [ ] Find string length with `len()`.
- [ ] Read characters using positive and negative indexes.
- [ ] Extract substrings using slicing.
- [ ] Search with `in`, `find()`, and `index()`.
- [ ] Check prefixes and suffixes.
- [ ] Split text into fields.
- [ ] Remove surrounding whitespace with `strip()`.
- [ ] Normalize simple text with `lower()` and `upper()`.
- [ ] Replace text and understand that a new string is returned.
- [ ] Join strings using a separator.
- [ ] Format readable messages with f-strings.
- [ ] Explain the assumptions and limitations of a simple log parser.
- [ ] Test empty and malformed input.

---

## Official Python documentation

1. [The Python Tutorial — Text](https://docs.python.org/3/tutorial/introduction.html#text) — introduces string literals, indexing, and slicing.
2. [Built-in Types — Text Sequence Type: str](https://docs.python.org/3/library/stdtypes.html#text-sequence-type-str) — reference for string behavior and methods.
3. [Common Sequence Operations](https://docs.python.org/3/library/stdtypes.html#common-sequence-operations) — documents operations such as indexing, slicing, membership, and length.
4. [String Methods](https://docs.python.org/3/library/stdtypes.html#string-methods) — documents methods including `find()`, `split()`, `join()`, `strip()`, and `replace()`.
5. [Formatted String Literals (f-strings)](https://docs.python.org/3/reference/lexical_analysis.html#f-strings) — describes f-string syntax.

The official documentation verifies the string operations and behavior. The log-line format and parser are deliberately simplified repository exercises.

---

**Previous:** [06 — Dictionaries](../06%20Dictionaries/README.md)

**Next:** [08 — Sorting](../08%20Sorting/README.md) — organize operational values and records into a useful order.
