# JSON

JSON (JavaScript Object Notation) is a text format commonly used to represent structured data. Python's standard library provides the `json` module for encoding Python objects as JSON and decoding JSON into Python objects.

> **Source note:** This topic is based on the official Python `json` documentation. The examples are teaching demonstrations created for this repository.

---

## 1. The mental model

Treat JSON as a data interchange boundary:

```text
JSON text
   ↓
parse / decode
   ↓
Python objects
   ↓
validate / transform
   ↓
Python objects
   ↓
serialize / encode
   ↓
JSON text
```

For DevOps automation, JSON commonly appears at boundaries such as API responses, configuration files, generated reports, and command output.

JSON is a **data format**, not a Python object model. Your program should validate data after decoding it when correctness matters.

---

## 2. Importing `json`

```python
import json
```

The standard library module provides functions and methods for working with JSON documents and JSON-compatible values.

---

## 3. JSON and Python types

The JSON data model maps naturally to several Python types.

A useful simplified mapping is:

| JSON | Python |
|---|---|
| object | `dict` |
| array | `list` |
| string | `str` |
| number | `int` / `float` |
| `true` | `True` |
| `false` | `False` |
| `null` | `None` |

The exact conversion behavior is documented by Python's `json` module.

Do not assume every Python object can automatically be represented as JSON.

---

## 4. `json.loads()` — decode a JSON string

Use `json.loads()` when JSON data is already available as a Python string or bytes-like input accepted by the API.

```python
import json

text = '{"service": "api", "replicas": 3}'
data = json.loads(text)

print(data["service"])
print(data["replicas"])
```

After decoding, `data` is a Python dictionary.

### Important distinction

```python
json.loads(...)
```

means **load from a string-like JSON representation**.

The `s` is a useful reminder that the source is a JSON string representation rather than a file object.

---

## 5. `json.dumps()` — encode Python data

Use `json.dumps()` to produce a JSON string from a Python object that can be serialized by the JSON encoder.

```python
import json

config = {
    "service": "api",
    "replicas": 3,
}

text = json.dumps(config)
print(text)
```

This is useful when an API client, command, or other interface expects JSON text.

---

## 6. Pretty-printing JSON

For human-readable output, `indent` can be supplied to `json.dumps()`.

```python
import json

config = {
    "service": "api",
    "replicas": 3,
}

print(json.dumps(config, indent=2))
```

Pretty formatting is useful for generated reports and troubleshooting output, but compact JSON may be preferable for machine-to-machine boundaries.

That choice is a repository/application design decision rather than a requirement of JSON itself.

---

## 7. Reading JSON from a file

The `json` module also provides `load()` and `dump()` for file-like objects.

```python
import json

with open("config.json", encoding="utf-8") as file:
    config = json.load(file)
```

The flow is:

```text
file
 ↓
json.load()
 ↓
Python object
```

Keep file handling and data validation conceptually separate:

```python
with open("config.json", encoding="utf-8") as file:
    config = json.load(file)

if not isinstance(config, dict):
    raise ValueError("configuration must be a JSON object")
```

---

## 8. Writing JSON to a file

Use `json.dump()` with a writable file object.

```python
import json

config = {
    "service": "api",
    "replicas": 3,
}

with open("config.json", "w", encoding="utf-8") as file:
    json.dump(config, file, indent=2)
```

The important sequence is:

```text
Python object
    ↓
json.dump()
    ↓
JSON file
```

Choose the file mode deliberately. `"w"` writes a new file or truncates an existing file before writing.

---

## 9. JSON decoding errors

Malformed JSON cannot be decoded successfully.

```python
import json

try:
    data = json.loads(text)
except json.JSONDecodeError as exc:
    raise ValueError("invalid JSON input") from exc
```

The official `json` documentation defines `JSONDecodeError` for JSON decoding failures.

Catch the narrow exception at the boundary where malformed input is expected.

Do not write:

```python
try:
    data = json.loads(text)
except Exception:
    pass
```

Silently discarding invalid configuration can cause later failures that are harder to diagnose.

---

## 10. Validate after parsing

Successful JSON parsing does not mean the data is correct for your application.

For example, this is valid JSON:

```json
{"replicas": "three"}
```

But an application expecting an integer replica count still needs validation.

```python
import json

config = json.loads('{"replicas": "three"}')

replicas = config.get("replicas")

if not isinstance(replicas, int):
    raise ValueError("replicas must be an integer")
```

The general DevOps pattern is:

```text
parse
  ↓
validate structure
  ↓
validate values
  ↓
perform action
```

JSON parsing and application-level schema validation are different responsibilities.

---

## 11. Missing keys

A decoded JSON object becomes a Python dictionary, so normal dictionary access rules apply.

```python
service = config["service"]
```

A missing key raises `KeyError`.

For optional configuration, `get()` can provide a default:

```python
region = config.get("region", "default-region")
```

Whether a field is required or optional is an application design decision.

---

## 12. Nested JSON

JSON objects and arrays can be nested.

```python
config = {
    "service": {
        "name": "api",
        "ports": [8080, 8443],
    }
}
```

Access follows the decoded Python structure:

```python
name = config["service"]["name"]
ports = config["service"]["ports"]
```

For deeply nested external data, validate the structure before accessing multiple levels directly.

---

## 13. JSON and APIs

Many APIs exchange JSON request and response bodies.

The JSON responsibility is the data conversion boundary:

```text
HTTP response body
       ↓
JSON decoding
       ↓
Python data
       ↓
validation
       ↓
automation logic
```

The HTTP transport itself belongs to the later **HTTP/REST** topic.

Do not treat successful JSON parsing as proof that an API operation succeeded. Transport status, API-level fields, and application-specific validation are separate concerns.

---

## 14. JSON and environment configuration

Environment variables are strings, while JSON can represent structured values.

A program can receive a JSON document through an environment variable and decode it:

```python
import json
import os

raw_config = os.environ["APP_CONFIG_JSON"]
config = json.loads(raw_config)
```

This can be useful at certain integration boundaries, but large or sensitive configuration values should be handled according to the surrounding platform's configuration and secret-management practices.

Do not log configuration blindly just because it is JSON; it may contain credentials or other sensitive values.

---

## 15. Serialization limits

Not every Python object is JSON serializable by default.

For example, a custom class instance cannot simply be assumed to serialize:

```python
import json

class Server:
    pass

json.dumps(Server())
```

When an object is unsupported, the encoder raises `TypeError` unless an appropriate serialization strategy is provided.

A practical approach is to convert application objects into JSON-compatible structures at the serialization boundary.

For example:

```python
server = {
    "name": "api-01",
    "enabled": True,
}

text = json.dumps(server)
```

Keep serialization logic explicit rather than depending on accidental object representations.

---

## 16. Sorting JSON object keys

`json.dumps()` supports `sort_keys=True`.

```python
import json

config = {
    "z": 1,
    "a": 2,
}

print(json.dumps(config, sort_keys=True))
```

Deterministic key ordering can make generated JSON easier to compare in reviews or tests.

This is a repository/application design choice, not a requirement for valid JSON.

---

## 17. JSON and command-line automation

JSON is often useful as an intermediate representation between automation steps.

A practical pattern is:

```text
external command / API
        ↓
JSON
        ↓
Python parsing
        ↓
validation
        ↓
filter / transform
        ↓
action or report
```

Keep parsing separate from side effects.

For example, a function can accept already-decoded data and return the subset needed by the next stage:

```python
def enabled_services(data):
    services = data.get("services", [])
    return [item for item in services if item.get("enabled") is True]
```

This makes the transformation easier to test without invoking external systems.

---

## 18. Common mistakes

### Mistake 1: confusing `load()` and `loads()`

```text
load  → file-like object
loads → string-like JSON data
```

### Mistake 2: assuming parsing means validation

Valid JSON can still contain the wrong structure or values.

### Mistake 3: catching every exception

Catch the specific parsing exception at the input boundary when appropriate.

### Mistake 4: logging sensitive JSON

JSON is only a format. It can contain passwords, tokens, keys, or other sensitive information.

### Mistake 5: modifying decoded data without understanding aliases

Once decoded, JSON objects are normal Python containers. Normal Python mutation and aliasing rules apply.

### Mistake 6: treating JSON as a schema

JSON defines a data representation. Application-specific validation rules must be implemented separately unless a separate schema/validation system is being used.

### Mistake 7: assuming all Python objects serialize automatically

Use JSON-compatible structures or an explicit serialization strategy.

---

## 19. DevOps practice exercise

Build a small configuration loader:

```text
config.json
    ↓
load JSON
    ↓
verify top-level object
    ↓
verify required fields
    ↓
validate field types
    ↓
return configuration
```

Requirements:

1. Read the file with an explicit encoding.
2. Convert JSON into Python objects.
3. Produce a clear error for malformed JSON.
4. Require a `service` field.
5. Require `replicas` to be an integer.
6. Reject an invalid replica count.
7. Do not print the entire configuration automatically.

---

## 20. Knowledge checks

Before moving on, you should be able to answer:

1. What is the difference between `json.load()` and `json.loads()`?
2. What is the difference between `json.dump()` and `json.dumps()`?
3. What Python type represents a JSON object after decoding?
4. Why can valid JSON still be invalid application configuration?
5. What exception represents a JSON decoding error?
6. Why should configuration be validated after parsing?
7. Why should JSON configuration not automatically be printed to logs?
8. What happens when a Python object is not JSON serializable by default?
9. Why can deterministic key ordering be useful in generated JSON?
10. Why should JSON parsing be kept separate from external side effects?

---

## 21. Completion criteria

You are ready to move on when you can:

- decode JSON strings and files
- encode Python data as JSON
- distinguish `load`/`loads` and `dump`/`dumps`
- handle malformed JSON explicitly
- validate decoded structures
- work safely with nested data
- identify JSON serialization limits
- avoid exposing sensitive configuration in logs
- separate parsing, validation, transformation, and action

---

## 22. Source classification

### Verified facts

The behavior of Python's `json` module, including its loading/dumping interfaces, supported conversions, decoding exceptions, and serialization behavior, is based on the official Python documentation.

### Repository design decisions

The emphasis on `parse → validate → transform → act`, separating parsing from side effects, and using JSON in DevOps-oriented configuration workflows is the repository's teaching structure.

### Demonstrations

The code examples are intentionally small demonstrations. They are not claims about any particular cloud provider, API, CI/CD platform, or infrastructure tool.

---

## 23. Authoritative references

- Python `json` module: https://docs.python.org/3/library/json.html
- Python standard library: https://docs.python.org/3/library/
- Python tutorial — Input and Output: https://docs.python.org/3/tutorial/inputoutput.html
