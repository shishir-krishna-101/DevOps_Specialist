# Dictionaries

Dictionaries are Python's built-in mapping type. They associate keys with values and are especially useful in DevOps automation for configuration, structured command results, metadata, inventories, and parsed API responses.

## 1. Mental Model

Think of a dictionary as:

```text
key → value
```

Example:

```python
server = {
    "name": "web-01",
    "environment": "production",
    "port": 443,
}
```

The keys identify values. A key must satisfy the requirements for dictionary keys; in particular, keys must be hashable.

**Verified fact:** Python's dictionary is a mutable mapping type. Dictionary keys must be hashable.

**Repository design decision:** The examples in this file use strings for configuration-style keys because they are readable and common in automation code.

## 2. Creating Dictionaries

An empty dictionary:

```python
config = {}
```

With initial entries:

```python
config = {
    "region": "ap-south-1",
    "retries": 3,
    "enabled": True,
}
```

The `dict()` constructor can also create dictionaries:

```python
labels = dict(environment="production", team="platform")
```

When keys are identifiers, keyword arguments can be convenient. When keys are arbitrary strings or other hashable objects, use dictionary literals or other supported construction methods.

## 3. Accessing Values

Use a key inside square brackets:

```python
port = config["port"]
```

If the key does not exist, this raises `KeyError`.

For optional data, `get()` can provide a default:

```python
port = config.get("port")
```

Or:

```python
port = config.get("port", 80)
```

This is useful when a missing key is expected and you have a sensible default.

### `[]` vs `get()`

Use direct indexing when the key is required:

```python
region = config["region"]
```

Use `get()` when the key may legitimately be absent:

```python
timeout = config.get("timeout", 30)
```

Do not use `get()` merely to hide invalid configuration. If a required setting is missing, failing clearly can be safer than silently selecting an unintended default.

## 4. Adding and Updating Entries

Assigning to a key creates the entry if it does not exist and replaces its value if it does:

```python
config["timeout"] = 30
config["retries"] = 5
```

Multiple entries can be updated with `update()`:

```python
config.update({
    "timeout": 60,
    "retries": 4,
})
```

Existing keys are updated by assignment/update operations; new keys are added.

## 5. Checking for Keys

Use `in` to test dictionary membership:

```python
if "region" in config:
    print(config["region"])
```

Dictionary membership tests keys, not values.

```python
"region" in config       # checks keys
"ap-south-1" in config   # does not search values
```

If you need to search values explicitly, use `values()`.

## 6. Iterating Through a Dictionary

Iterating over a dictionary directly produces its keys:

```python
for key in config:
    print(key)
```

Use `keys()` when you want to make the intent explicit:

```python
for key in config.keys():
    print(key)
```

Use `values()` for values:

```python
for value in config.values():
    print(value)
```

Use `items()` when you need both:

```python
for key, value in config.items():
    print(key, value)
```

For most key/value processing, `items()` is the clearest choice.

## 7. Deleting Entries

`del` removes an entry:

```python
del config["retries"]
```

If the key is absent, this raises `KeyError`.

`pop()` removes a key and returns its value:

```python
timeout = config.pop("timeout")
```

A default can be supplied when absence is acceptable:

```python
timeout = config.pop("timeout", None)
```

`clear()` removes all entries:

```python
config.clear()
```

## 8. Dictionary Comprehensions

Dictionary comprehensions create dictionaries from an iterable:

```python
ports = [80, 443, 8080]
port_labels = {port: f"tcp-{port}" for port in ports}
```

They can include filtering:

```python
secure_ports = {
    port: "secure"
    for port in ports
    if port == 443
}
```

Use comprehensions when the transformation remains easy to read. If the expression becomes difficult to understand, use a normal loop or a function.

## 9. Nested Dictionaries

Automation data often has multiple levels:

```python
inventory = {
    "web": {
        "host": "web-01",
        "port": 443,
    },
    "worker": {
        "host": "worker-01",
        "port": 8080,
    },
}
```

Access nested values explicitly:

```python
web_port = inventory["web"]["port"]
```

For deeply nested structures, repeated direct indexing can become fragile. Validate the expected structure at the input boundary instead of spreading defensive checks throughout the application.

## 10. Configuration Modeling

Dictionaries are useful for representing configuration before it is validated and transformed into application-specific objects.

```python
config = {
    "environment": "production",
    "region": "ap-south-1",
    "retries": 3,
}

if config.get("environment") == "production":
    print("Use production policy")
```

A useful DevOps pattern is:

```text
raw input
   ↓
parse
   ↓
validate required keys and values
   ↓
normalize
   ↓
perform automation
```

The dictionary itself does not validate that its values are appropriate. Validation is an application responsibility.

## 11. Safe Access and Required Configuration

Avoid patterns that silently turn missing required configuration into an unrelated default.

Less explicit:

```python
region = config.get("region", "default-region")
```

If `region` is required, prefer explicit validation:

```python
if "region" not in config:
    raise ValueError("region is required")

region = config["region"]
```

The exact validation policy is a repository/application design decision. The important engineering principle is to distinguish optional settings from required settings.

## 12. Copying and Aliasing

Dictionaries are mutable, so assigning one dictionary to another name does not create an independent dictionary:

```python
original = {"retries": 3}
alias = original

alias["retries"] = 5

print(original["retries"])  # 5
```

Both names refer to the same dictionary object.

A shallow copy can be created with `copy()`:

```python
original = {"retries": 3}
copy_of_original = original.copy()
```

For nested mutable structures, a shallow copy still shares nested objects. Choose the copying strategy deliberately when modifying structured configuration.

## 13. Dictionary Views

Methods such as `keys()`, `values()`, and `items()` return view objects rather than ordinary lists.

```python
items = config.items()
```

If you specifically need a list snapshot, construct one:

```python
items = list(config.items())
```

Do not materialize collections unnecessarily when direct iteration is sufficient.

## 14. JSON and Dictionaries

JSON objects naturally map to Python dictionaries when JSON is decoded into Python objects.

For example, JSON such as:

```json
{
  "environment": "production",
  "retries": 3
}
```

can be represented in Python as:

```python
{
    "environment": "production",
    "retries": 3,
}
```

**Verified fact:** Python's `json` module provides JSON encoding and decoding facilities. JSON data types and Python data types are not identical, so serialization boundaries should be treated explicitly.

The detailed JSON workflow is covered in the later **JSON** topic.

## 15. Choosing Between Dictionary and Other Collections

Use a dictionary when you need key/value association.

Use a list when order and positional access are the main concerns.

Use a set when membership and uniqueness are the main concerns.

Use a tuple when an immutable sequence is appropriate.

For specialized workloads, Python's standard library also provides mapping and collection types such as `defaultdict` and `Counter`.

The choice should follow the data model rather than forcing every collection into a dictionary.

## 16. DevOps Automation Patterns

### Resource metadata

```python
resource = {
    "name": "api-01",
    "environment": "staging",
    "owner": "platform",
}
```

### Command results

```python
result = {
    "command": "example",
    "return_code": 0,
    "stdout": "ok",
}
```

This is a teaching model; actual subprocess behavior should be handled through Python's `subprocess` APIs rather than invented command-result conventions.

### Inventory lookup

```python
hosts = {
    "web": "10.0.0.10",
    "worker": "10.0.0.20",
}

host = hosts["web"]
```

### Aggregating structured data

```python
status_counts = {}

for status in ["healthy", "healthy", "degraded"]:
    status_counts[status] = status_counts.get(status, 0) + 1
```

This pattern is useful when you need a simple frequency mapping. For more specialized counting, `collections.Counter` is available in the standard library.

## 17. Common Mistakes

### Mistake 1: Assuming a key exists

```python
value = config["missing-key"]
```

Know whether absence is an error or an expected condition.

### Mistake 2: Confusing keys and values

```python
if "production" in config:
    ...
```

This checks keys. It does not search values.

### Mistake 3: Accidentally sharing a mutable dictionary

```python
copy = original
```

This creates another reference to the same object, not an independent dictionary.

### Mistake 4: Overusing `get()`

Using `get()` everywhere can hide malformed required configuration. Use it intentionally for optional data.

### Mistake 5: Building deeply nested unvalidated data

Large nested dictionaries can become difficult to reason about. Validate external data at boundaries and consider dedicated data models when the structure becomes complex.

### Mistake 6: Mutating while iterating over a dictionary

Avoid changing a dictionary's size while iterating directly over it:

```python
for key in config:
    del config[key]  # unsafe pattern
```

If you need to remove entries, iterate over an appropriate snapshot or build the desired result separately.

## 18. Practical Exercises

### Exercise 1 — Environment configuration

Create a dictionary containing an environment name, region, retry count, and deployment flag. Read the values and validate the required fields.

### Exercise 2 — Inventory lookup

Create a dictionary mapping service names to hostnames. Write code that reports whether a requested service exists before accessing it.

### Exercise 3 — Status aggregation

Given a list of service statuses, build a dictionary containing the number of occurrences of each status.

### Exercise 4 — Filter configuration

Given a dictionary of configuration values, create a new dictionary containing only entries whose keys begin with a chosen prefix.

### Exercise 5 — Nested inventory

Represent several environments as nested dictionaries. Write a function that retrieves a service endpoint while handling missing environments and services explicitly.

### Exercise 6 — Safe update

Write a function that updates optional configuration fields but refuses to continue when required fields are missing.

## 19. Knowledge Check

Before moving on, you should be able to answer:

1. What problem does a dictionary solve?
2. What is the difference between `config["key"]` and `config.get("key")`?
3. What does `in` test when used with a dictionary?
4. What is the difference between `keys()`, `values()`, and `items()`?
5. How do you add or update an entry?
6. How can you remove an entry while retrieving its value?
7. Why does `alias = original` not create a copy?
8. What does a shallow dictionary copy mean for nested mutable values?
9. When is a dictionary comprehension useful?
10. Why should required configuration be validated explicitly?

## 20. Completion Criteria

You are ready to move on when you can:

- create and update dictionaries confidently;
- access required and optional values appropriately;
- iterate over keys, values, and key/value pairs;
- add, remove, and transform entries;
- write readable dictionary comprehensions;
- explain aliasing versus copying;
- work with nested configuration safely;
- model common DevOps metadata and inventory data;
- distinguish dictionary behavior from application-level validation;
- choose an appropriate collection type for the data being modeled.

## 21. Source Notes

### Verified facts

The Python language and standard-library behavior described here is based on the official Python documentation.

### Repository design decisions

The DevOps examples, learning sequence, exercises, and configuration-validation patterns are teaching choices for this repository. They are not statements about how a particular cloud provider, Kubernetes, CI system, or automation platform must represent its data.

### Authoritative references

- Python Tutorial — Data Structures: https://docs.python.org/3/tutorial/datastructures.html#dictionaries
- Python Standard Library — Mapping Types — `dict`: https://docs.python.org/3/library/stdtypes.html#mapping-types-dict
- Python Standard Library — `collections`: https://docs.python.org/3/library/collections.html
- Python Standard Library — `json`: https://docs.python.org/3/library/json.html
