# Variables and Basic Data Types

Variables and basic data types are the foundation of Python automation. In DevOps work, you constantly move values between configuration, files, command output, APIs, validation logic, and operational actions.

The goal is not to memorize every built-in type. Learn to recognize what kind of value you have, what operations it supports, whether it can be changed in place, and what assumptions your automation is making about it.

---

## 1. The mental model: names, objects, and values

Python's data model describes program data in terms of **objects**. Every object has an identity, a type, and a value.

A variable name is a name bound to an object.

```python
service = "api"
replicas = 3
healthy = True
```

A useful mental model is:

```text
name
  ↓
binds to
  ↓
object
  ├── type
  ├── identity
  └── value
```

This matters because assigning another name does not necessarily create a new object.

```python
servers = ["web-1", "web-2"]
backup = servers
```

Both names refer to the same list object. If the list is changed through one name, the other name observes that change.

```python
servers.append("web-3")

print(backup)
# ['web-1', 'web-2', 'web-3']
```

Do not think of Python variables as boxes that always contain independent copies of values. Think in terms of names referring to objects.

---

## 2. Creating and assigning variables

Use the assignment operator `=` to bind a name to a value.

```python
environment = "production"
port = 8080
healthy = True
```

Python does not require you to declare a variable's type separately before assigning a value.

You can inspect a value's type with `type()` while learning or troubleshooting:

```python
value = 8080
print(type(value))
```

The result is a type object representing `int`.

### Reassignment

A name can later be bound to another object:

```python
status = "starting"
status = "healthy"
```

The name `status` now refers to the new value.

### Multiple assignment

Python supports assigning several names in one statement:

```python
service, environment = "api", "production"
```

You can also unpack an iterable:

```python
server, port = ("web-1", 8080)
```

The number and structure of the values must match the assignment pattern.

---

## 3. Choosing useful names

Good names communicate what an automation value represents.

Prefer:

```python
retry_limit = 3
service_name = "api"
environment = "production"
```

Over vague names:

```python
x = 3
a = "api"
v = "production"
```

For DevOps scripts, clear names reduce operational mistakes because values often represent real infrastructure state or deployment policy.

### Naming principle

A reader should be able to understand the role of a value without tracing the entire program.

```python
request_timeout_seconds = 10
maximum_retries = 3
configuration_path = "config.json"
```

These names expose intent better than `t`, `n`, and `p`.

---

## 4. Integers: `int`

Python's `int` type represents integers.

```python
replica_count = 3
port = 8080
retry_limit = 5
```

Python integers have unlimited precision, subject to available memory.

Common DevOps uses include:

- replica counts;
- retry limits;
- ports;
- counters;
- sizes expressed as integer units;
- exit-status values when represented as integers.

Basic arithmetic:

```python
attempt = 2
next_attempt = attempt + 1
```

Integer division has two different forms that are easy to confuse:

```python
print(7 / 2)   # 3.5
print(7 // 2)  # 3
```

`/` produces a division result as a floating-point number, while `//` performs floor division.

The `%` operator gives the remainder:

```python
print(7 % 2)  # 1
```

---

## 5. Floating-point numbers: `float`

`float` represents floating-point numbers.

```python
cpu_load = 0.75
latency_seconds = 1.5
```

They are useful for measurements and ratios, but floating-point representation has precision limitations.

For example, do not assume that every decimal fraction can be represented exactly as a binary floating-point value.

If exact decimal behavior is required for a particular problem, investigate an appropriate numeric representation rather than assuming `float` provides decimal arithmetic.

### DevOps example

A monitoring calculation might produce:

```python
successful = 97
requests = 100
success_rate = successful / requests
```

The result is a floating-point value.

Do not confuse a measurement's display precision with the underlying numeric representation.

---

## 6. Booleans: `bool`

Python has two Boolean values:

```python
True
False
```

They are commonly used for state and decision logic.

```python
service_healthy = True
maintenance_mode = False
```

Use them directly in conditions:

```python
if service_healthy:
    print("Continue")
```

Python's `bool` type is a subtype of `int`. Although this is part of Python's type system, do not use that relationship as a reason to mix Boolean state and numeric policy carelessly.

Prefer code whose intent is obvious:

```python
healthy = True
```

rather than encoding application state as arbitrary integers.

---

## 7. `None`: absence of a value

`None` is Python's value used to represent the absence of a value in many situations.

```python
result = None
```

A common check is:

```python
if result is None:
    print("No result")
```

Use `is None` rather than `== None` for this identity check.

`None` is useful when a function needs to distinguish between:

```text
no value was supplied
```

and a real value such as:

```text
0
""
False
[]
```

Those values have different meanings even though several of them are false in Boolean contexts.

---

## 8. Strings: `str`

Strings represent text.

```python
service = "api"
environment = "production"
```

Strings are used heavily in automation for:

- service names;
- hostnames;
- file paths represented as text;
- command arguments;
- environment-variable values;
- API fields;
- log messages;
- identifiers.

Strings can be indexed and sliced:

```python
name = "python"

print(name[0])
print(name[:3])
```

Strings are immutable. Operations that appear to modify a string produce another string value rather than changing the existing string in place.

```python
name = "api"
name = name.upper()
```

The second assignment binds `name` to the new string.

### Formatting text

F-strings are useful when constructing readable messages:

```python
service = "api"
environment = "production"

message = f"Deploying {service} to {environment}"
```

Keep operational messages clear, and do not place secrets into messages simply because they are available as variables.

---

## 9. Lists: `list`

Lists represent ordered collections and are mutable.

```python
services = ["api", "worker", "scheduler"]
```

You can change a list in place:

```python
services.append("frontend")
```

Useful operations include:

```python
len(services)
services[0]
services[-1]
```

Lists are useful when order matters and the collection may need to change during execution.

### Aliasing warning

```python
services = ["api", "worker"]
backup = services

services.append("scheduler")

print(backup)
```

`backup` refers to the same list object. If you need an independent shallow copy for the use case, create one deliberately:

```python
backup = services.copy()
```

The correct choice depends on whether you want shared mutation or independent list structure.

---

## 10. Tuples: `tuple`

Tuples are immutable sequences.

```python
server = ("web-1", 8080)
```

They can be useful when a fixed grouping of values is appropriate.

Tuple unpacking is common:

```python
host, port = server
```

A tuple's immutability applies to the tuple's sequence structure. If it contains a mutable object, that contained object can still be changed.

For example:

```python
record = ("web-1", ["api", "worker"])
record[1].append("scheduler")
```

The tuple itself remains the same sequence of references, while the referenced list changes.

This is an important distinction when reasoning about configuration and shared data.

---

## 11. Sets: `set`

A set is a collection designed around unique elements.

```python
environments = {"dev", "test", "prod"}
```

Sets are useful when uniqueness or membership matters more than sequence order.

For example:

```python
required = {"api", "worker", "scheduler"}
deployed = {"api", "worker"}

missing = required - deployed
```

This can make certain inventory and comparison operations clearer than manually searching a list.

An empty set is created with `set()`, not `{}`:

```python
empty_set = set()
empty_dict = {}
```

`{}` creates an empty dictionary.

---

## 12. Dictionaries: `dict`

Dictionaries store key/value pairs.

```python
service = {
    "name": "api",
    "environment": "production",
    "healthy": True,
}
```

They are extremely common in automation because structured configuration and decoded JSON data naturally map to key/value structures.

Access a required key with subscription syntax:

```python
name = service["name"]
```

If a key may be absent, `.get()` can provide a default:

```python
port = service.get("port", 8080)
```

Iterate through key/value pairs with `.items()`:

```python
for key, value in service.items():
    print(key, value)
```

A dictionary is mutable:

```python
service["healthy"] = False
```

Be explicit about whether missing data should be treated as an error or as a default. Silent defaults can hide malformed configuration.

---

## 13. Truth values and conditions

Python objects can be tested for truth value in conditions and Boolean expressions.

Common false values include:

- `None`;
- `False`;
- numeric zero values;
- empty strings;
- empty lists;
- empty tuples;
- empty dictionaries;
- empty sets.

For example:

```python
services = []

if not services:
    print("No services configured")
```

This is concise, but make sure the meaning is correct for the application.

For example, these may represent different states in an automation system:

```python
value = None
value = 0
value = ""
value = []
```

All can be false in a condition, but they do not necessarily mean the same thing operationally.

---

## 14. `==` versus `is`

Use `==` for value comparison:

```python
environment = "production"

if environment == "production":
    print("Production deployment")
```

Use `is` for object identity checks:

```python
result = None

if result is None:
    print("No result")
```

Do not use `is` as a general substitute for `==`.

The distinction is important because two objects can have equal values without being the same object.

```python
first = ["api"]
second = ["api"]

print(first == second)  # True
print(first is second)  # False
```

The exact identity of immutable values should not be inferred from examples where an implementation may reuse objects. Use value comparison when you mean value comparison.

---

## 15. Mutable versus immutable objects

Python's data model distinguishes mutable and immutable objects.

Common immutable types include:

- numbers;
- strings;
- tuples.

Common mutable types include:

- lists;
- dictionaries;
- sets.

The practical difference is whether the object's value can be changed in place.

### Mutable example

```python
servers = ["web-1"]
servers.append("web-2")
```

The list object is changed.

### Immutable example

```python
name = "web"
name = name + "-1"
```

The original string is not modified. The expression produces another string, and the name is rebound to that value.

### Why DevOps engineers should care

Mutable objects can create surprising shared-state bugs:

```python
servers = ["web-1"]
current = servers

current.append("web-2")
```

If the code assumes `servers` and `current` are independent, the behavior is wrong.

When passing collections between functions, decide whether the function should:

- mutate the supplied object;
- return a new object;
- or treat the input as read-only by convention.

Make that contract clear.

---

## 16. Type conversion

Python provides constructors and conversion functions for many common types.

```python
port = int("8080")
ratio = float("0.75")
text = str(8080)
```

Conversion can fail when the input does not represent a valid value for the requested type:

```python
port = int("not-a-port")
```

This raises an exception rather than silently producing an arbitrary integer.

For automation, validate external input before using it as operational data.

### Do not confuse strings and numbers

```python
port = "8080"
```

is text, not an integer.

This distinction matters when comparing, calculating, serializing, or passing values to APIs and subprocesses.

---

## 17. Type checks and design

`type()` can help during exploration:

```python
value = 42
print(type(value))
```

But production logic should not automatically become a collection of exact-type checks.

First ask what behavior the code actually requires.

For example, if a function needs an object that supports iteration, design around that requirement rather than assuming the only valid input is a specific concrete collection.

More advanced Python topics such as protocols, type hints, and data-model behavior are covered later in the curriculum.

---

## 18. DevOps example: representing service configuration

A small configuration object might use several basic types together:

```python
service = {
    "name": "api",
    "port": 8080,
    "enabled": True,
    "replicas": 3,
    "environment": "production",
}
```

A validation function can check the values before an operational action:

```python
def validate_service(config):
    if not config.get("name"):
        raise ValueError("service name is required")

    if not isinstance(config.get("port"), int):
        raise TypeError("port must be an integer")

    if config["port"] < 1 or config["port"] > 65535:
        raise ValueError("port is outside the valid range")

    if config.get("replicas", 0) < 0:
        raise ValueError("replicas cannot be negative")
```

The important design is the sequence:

```text
external data
     ↓
represent as Python values
     ↓
validate assumptions
     ↓
perform operational action
```

Do not assume that data decoded from a file or API is automatically valid just because Python can represent it.

---

## 19. Common mistakes

### Mistake 1: Treating every variable as an independent container

Names can refer to the same object. Mutable objects make this especially important.

### Mistake 2: Using `is` for normal value comparison

Use `==` for values and `is` for identity checks such as `value is None`.

### Mistake 3: Assuming empty means the same thing as missing

`None`, `""`, `0`, and `[]` can all be false, but they can have different operational meanings.

### Mistake 4: Treating numeric-looking text as a number

`"8080"` and `8080` are different values with different types.

### Mistake 5: Mutating shared collections unexpectedly

If two names reference the same list or dictionary, an in-place update through one name is visible through the other.

### Mistake 6: Using a mutable default as shared state

Be careful when designing function defaults. Mutable default arguments are evaluated once when the function is defined, not once per call. This behavior is covered in more detail in the Complete Python functions topics.

### Mistake 7: Hiding configuration problems with defaults

A default can be useful, but silently replacing missing required configuration with a default can turn a configuration error into an operational error later.

---

## 20. DevOps exercises

### Exercise 1 — Model a service

Create variables for:

- service name;
- environment;
- port;
- replica count;
- health state.

Print each value and its type.

### Exercise 2 — Shared mutation

Create a list of services and assign it to a second name.

Append an item through the second name.

Observe the first name.

Then create a shallow copy and repeat the experiment. Explain the difference.

### Exercise 3 — Configuration validation

Create a dictionary representing a service configuration.

Write a function that rejects:

- a missing service name;
- a non-integer port;
- a port outside the valid port range;
- a negative replica count.

### Exercise 4 — Missing versus empty

Create examples using:

```python
None
""
0
[]
```

Test each in an `if` statement.

Then explain why a DevOps configuration system might need to distinguish these values explicitly.

### Exercise 5 — Value versus identity

Create two separate lists containing the same values.

Compare them with both `==` and `is`.

Explain why the results differ.

---

## 21. Knowledge check

You should be able to answer these without looking at the notes:

1. What is a Python variable name actually bound to?
2. What are an object's identity, type, and value?
3. Which common built-in collection types are mutable?
4. Why does `is` differ from `==`?
5. Why is `value is None` preferred to `value == None`?
6. What is the difference between `7 / 2` and `7 // 2`?
7. Why is `"8080"` different from `8080`?
8. What does `None` normally represent?
9. Why can assigning one list to another name cause shared-state behavior?
10. Why should configuration be validated before an operational action?

---

## 22. Completion criteria

You have completed this topic when you can:

- explain names, objects, types, and values;
- create and reassign variables clearly;
- choose appropriate names for automation data;
- use `int`, `float`, `bool`, `None`, and `str` correctly;
- work with lists, tuples, sets, and dictionaries at a basic level;
- explain mutable versus immutable objects;
- distinguish value equality from object identity;
- recognize false values in Boolean contexts;
- convert external text into appropriate Python types deliberately;
- identify shared mutable state;
- validate basic configuration data before using it operationally.

---

## 23. Source notes

### Verified facts

The official Python documentation establishes the object model, identity/type/value model, mutability concepts, built-in types, truth-value testing, comparisons, numeric types, and basic assignment behavior used in this topic.

### Repository design decisions

The DevOps examples emphasize explicit configuration validation, clear variable names, controlled mutation, and careful handling of external data. These are learning and repository design choices, not requirements imposed by Python.

### Authoritative references

- Python Tutorial — An Informal Introduction to Python: https://docs.python.org/3/tutorial/introduction.html
- Python Standard Library — Built-in Types: https://docs.python.org/3/library/stdtypes.html
- Python Language Reference — Data Model: https://docs.python.org/3/reference/datamodel.html
- Python Language Reference — Expressions: https://docs.python.org/3/reference/expressions.html
