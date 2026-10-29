# Dataclasses and Type Hints

## Why this topic matters

As Python automation grows, scripts often move from loose dictionaries and positional values toward structured data models. Dataclasses provide a concise way to define classes primarily used to store data, while type hints communicate intended types to readers and tools.

For DevOps automation, this combination is useful when representing configuration, deployment parameters, resource metadata, command results, or other structured information.

> **Verified fact:** Python's `dataclasses` module provides a `@dataclass` decorator that can add generated special methods to classes based on annotated fields.
>
> **Verified fact:** Python type annotations are not, by themselves, a runtime type-checking system. Python's `typing` module primarily supports type hints and tools that consume them.
>
> **Repository design decision:** This guide uses infrastructure and automation examples because the surrounding repository is DevOps-focused.

---

## 1. A dataclass is a structured data model

A normal class can represent structured data:

```python
class Server:
    def __init__(self, name, environment, port):
        self.name = name
        self.environment = environment
        self.port = port
```

A dataclass can express the same basic model more concisely:

```python
from dataclasses import dataclass


@dataclass
class Server:
    name: str
    environment: str
    port: int
```

The annotations describe the fields, and `@dataclass` can generate methods such as an initializer and representation methods according to its configuration.

This is especially useful when the class primarily represents data rather than complex behavior.

---

## 2. Fields and annotations

A dataclass field is normally declared with an annotation:

```python
@dataclass
class Deployment:
    application: str
    environment: str
    replicas: int
```

Instances can then be created with named values:

```python
deployment = Deployment(
    application="api",
    environment="production",
    replicas=3,
)
```

The annotation does not automatically convert or validate the supplied value.

For example, a type annotation such as `replicas: int` communicates that an integer is intended. It does not mean that Python will automatically reject every other runtime value.

---

## 3. Defaults

Dataclass fields can have default values:

```python
@dataclass
class RetryPolicy:
    attempts: int = 3
    timeout_seconds: int = 30
```

A value can be overridden when constructing the object:

```python
policy = RetryPolicy(attempts=5)
```

As with normal Python function parameters, required values should not be placed after fields that already have defaults when doing so would create an invalid generated initializer.

---

## 4. `field()` and mutable defaults

The `field()` helper provides additional control over dataclass fields.

A common pattern is using `default_factory` when each instance needs its own mutable object:

```python
from dataclasses import dataclass, field


@dataclass
class Job:
    name: str
    tags: list[str] = field(default_factory=list)
```

Each `Job` instance gets a list created by the factory rather than sharing one mutable list between instances.

This is an important distinction in automation code because shared mutable state can cause one job, deployment, or resource object to unexpectedly affect another.

> **Common mistake:** Do not treat `default_factory=list` as interchangeable with creating one shared list and assigning it as a class-level default.

---

## 5. Generated methods

The `@dataclass` decorator can generate methods based on its parameters and the class fields.

For example:

```python
@dataclass
class Server:
    name: str
    port: int
```

A useful representation is available when inspecting an instance:

```python
server = Server("web-01", 8080)
print(server)
```

Dataclasses can also participate in equality comparisons according to their configured behavior.

Do not assume every possible method is generated in every configuration. Consult the official `dataclasses` documentation when changing options such as `eq`, `order`, `frozen`, or representation-related parameters.

---

## 6. Immutable-style models with `frozen`

For data that should not normally be changed after construction, a dataclass can be configured as frozen:

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Endpoint:
    host: str
    port: int
```

The frozen configuration prevents ordinary assignment to fields after initialization.

This can be useful for values that conceptually represent immutable configuration or identity.

> **Important:** `frozen=True` should not be described as making every object reachable from the instance deeply immutable. The behavior applies to assignment through the dataclass's fields; nested mutable objects can still have their own mutability.

---

## 7. Keyword-only fields

Current Python dataclasses support keyword-only fields through the documented dataclass field mechanisms.

For example, when keyword-only construction is appropriate:

```python
from dataclasses import dataclass


@dataclass(kw_only=True)
class Connection:
    host: str
    port: int
```

Construction then uses explicit field names:

```python
connection = Connection(host="db.internal", port=5432)
```

This style can make configuration-heavy code easier to read because important values are labeled at the call site.

Always check the Python version targeted by a project before relying on version-specific dataclass parameters.

---

## 8. Dataclass inheritance

Dataclasses can participate in inheritance.

A simple example:

```python
@dataclass
class Resource:
    name: str


@dataclass
class VM(Resource):
    size: str
```

The derived dataclass includes the inherited data fields as part of the generated model.

Inheritance can be useful when several resource models share stable attributes, but it should not be used merely to avoid writing a few repeated fields. Composition can be clearer when models represent independent concepts.

---

## 9. Converting dataclass instances

The `dataclasses` module provides utilities for working with dataclass instances.

`asdict()` can recursively convert a dataclass instance to a dictionary representation:

```python
from dataclasses import asdict


@dataclass
class Service:
    name: str
    port: int


service = Service("api", 8080)
print(asdict(service))
```

`astuple()` provides a corresponding tuple representation.

`replace()` creates a new dataclass instance with selected fields replaced:

```python
from dataclasses import replace

updated = replace(service, port=9090)
```

These utilities can be useful at boundaries such as serialization or transformation, but the resulting representation should not automatically be treated as an API schema.

---

## 10. Type hints communicate intent

Type annotations can be attached to variables, function parameters, return values, attributes, and other supported Python constructs.

Example:

```python
def build_command(service: str, port: int) -> list[str]:
    return ["run-service", service, "--port", str(port)]
```

The annotations communicate that:

- `service` is expected to be a string.
- `port` is expected to be an integer.
- the function is intended to return a list of strings.

This makes interfaces easier to understand and gives static-analysis/type-checking tools information they can inspect.

---

## 11. Built-in generic types

Modern Python supports parameterized built-in collection types in annotations:

```python
names: list[str]
ports: set[int]
metadata: dict[str, str]
```

Function signatures can use the same form:

```python
def group_services(names: list[str]) -> dict[str, int]:
    ...
```

Prefer annotations that describe the contract clearly rather than adding types solely to make a file look more sophisticated.

---

## 12. Optional values and unions

A value that may have more than one type can be represented with a union type.

Modern Python supports the `|` syntax:

```python
def get_port(service: str) -> int | None:
    ...
```

This communicates that the function may return an integer or `None`.

The `typing` module also contains tools for expressing more complex type relationships. Use the form that matches the Python versions supported by the project.

---

## 13. `Any` means reduced type information

`Any` is available from `typing`:

```python
from typing import Any

payload: Any = load_payload()
```

`Any` is intentionally permissive. It can be appropriate at genuinely dynamic boundaries, but using it everywhere removes useful information from the type system.

For DevOps automation, a better approach is often to keep dynamic data at a narrow boundary and convert it into a more explicit model afterward.

---

## 14. Callable types

Functions can also be described by type hints.

For example:

```python
from collections.abc import Callable


def run_with_retry(action: Callable[[], bool]) -> bool:
    return action()
```

This communicates that `action` is expected to be callable without arguments and produce a boolean result.

The exact callable signature should reflect the function's actual contract.

---

## 15. Type hints are not automatic validation

Consider:

```python
def scale_replicas(replicas: int) -> int:
    return replicas * 2
```

The annotation communicates intent. It does not itself provide a general runtime validation layer.

If input comes from an untrusted or loosely typed boundary such as JSON, environment variables, command-line arguments, or external APIs, validate and normalize that input explicitly.

A useful architecture is:

```text
External input
      ↓
Parse / validate
      ↓
Structured Python model
      ↓
Automation logic
      ↓
External action
```

This keeps boundary validation separate from the internal representation used by the automation code.

---

## 16. Reading annotations with `get_type_hints`

Python provides `typing.get_type_hints()` for retrieving evaluated type hints for supported Python objects.

Example:

```python
from typing import get_type_hints


def deploy(name: str, replicas: int) -> bool:
    return True


print(get_type_hints(deploy))
```

This can be useful for tooling or introspection.

Do not treat the result as a replacement for explicit runtime validation of external input.

---

## 17. Dataclasses for DevOps configuration

A dataclass can make a configuration model easier to read:

```python
@dataclass
class DeploymentConfig:
    application: str
    environment: str
    replicas: int = 1
    dry_run: bool = False
```

A loader can convert external input into this model after validation:

```text
YAML / JSON / CLI / environment
              ↓
       Parse external data
              ↓
       Validate assumptions
              ↓
      DeploymentConfig
              ↓
       Deployment logic
```

> **Repository design decision:** This repository treats dataclasses as one possible internal model. They are not mandatory for every automation script.

---

## 18. Keep serialization boundaries explicit

A dataclass instance is a Python object, not automatically a JSON document, YAML document, API request, or Kubernetes resource.

If an external system requires a particular schema, the conversion should be explicit and should follow that system's authoritative specification.

For example:

```python
@dataclass
class DeploymentConfig:
    application: str
    replicas: int


def to_payload(config: DeploymentConfig) -> dict[str, object]:
    return {
        "application": config.application,
        "replicas": config.replicas,
    }
```

The explicit conversion makes the boundary visible and testable.

---

## 19. `ClassVar` for class-level values

When a dataclass contains a value intended to be a class variable rather than a dataclass field, `ClassVar` can communicate that distinction to dataclass processing and type-checking tools.

Example:

```python
from typing import ClassVar


@dataclass
class Service:
    name: str
    default_port: ClassVar[int] = 8080
```

The distinction matters when a class has metadata shared by all instances rather than per-instance state.

---

## 20. `InitVar` for initialization-only input

The `dataclasses` module also provides `InitVar` for values that can be supplied during initialization without becoming normal stored dataclass fields.

This can be useful when initialization requires an input that is used to construct or validate the instance's stored state.

Use it only when that distinction improves the model. A normal field is often clearer.

---

## 21. Dataclasses versus ordinary dictionaries

A dictionary is often the simplest representation for genuinely dynamic data:

```python
metadata = {
    "region": "ap-south-1",
    "owner": "platform",
}
```

A dataclass is useful when the structure is stable and has a meaningful model:

```python
@dataclass
class ResourceMetadata:
    region: str
    owner: str
```

A practical rule is:

```text
Dynamic / unknown shape → dictionary
Stable / known structure → explicit model
```

This is a repository design heuristic, not a Python language requirement.

---

## 22. Common mistakes

### Mistake 1: Assuming annotations validate values

```python
replicas: int = "three"
```

A type annotation does not automatically perform general runtime validation.

### Mistake 2: Sharing mutable defaults

Prefer a documented factory for per-instance mutable state:

```python
items: list[str] = field(default_factory=list)
```

### Mistake 3: Treating a dataclass as an external schema

A dataclass does not automatically guarantee that an API, configuration file, or cloud service accepts its fields.

### Mistake 4: Using `Any` everywhere

This can make a codebase harder to understand and reduces the value of type information.

### Mistake 5: Over-modeling

Not every short script needs multiple dataclasses and a complex type hierarchy.

### Mistake 6: Ignoring the supported Python version

Typing and dataclass features have evolved over Python releases. Verify the target Python version before using newer syntax or parameters.

### Mistake 7: Hiding boundary validation

If data comes from an external system, make parsing and validation explicit instead of assuming that type hints guarantee correct input.

---

## 23. DevOps exercises

### Exercise 1 — Deployment model

Create a `DeploymentConfig` dataclass containing:

- application name
- environment
- replicas
- dry-run flag

Instantiate it and print the object.

### Exercise 2 — Mutable field

Create a dataclass representing a job with a list of tags. Use `default_factory` and verify that two instances do not share the same list.

### Exercise 3 — Typed function

Write a function that accepts a list of service names and returns a dictionary mapping each name to an integer port.

### Exercise 4 — External boundary

Take a dictionary representing deployment input and explicitly convert it into a dataclass after checking that required values are present and have acceptable types.

### Exercise 5 — Immutable configuration

Create a frozen dataclass for a connection endpoint. Try to modify one of its fields and observe the behavior described by the official documentation.

### Exercise 6 — Serialization boundary

Write an explicit function that converts a dataclass into the dictionary shape required by your own demonstration program.

---

## 24. Knowledge checks

You should be able to answer these without looking at the file:

1. What problem does `@dataclass` solve?
2. What information does a type annotation communicate?
3. Do type hints automatically validate runtime values?
4. Why is `default_factory` useful for mutable fields?
5. What is the purpose of `frozen=True`?
6. When would a dictionary be preferable to a dataclass?
7. Why should external input be validated before becoming an internal model?
8. What does `Any` communicate?
9. What does `get_type_hints()` provide?
10. Why should serialization to an external API schema remain explicit?

---

## 25. Completion checklist

You are ready to move on when you can:

- [ ] Define a dataclass with annotated fields.
- [ ] Use default values safely.
- [ ] Use `field(default_factory=...)` for per-instance mutable values.
- [ ] Explain generated dataclass methods at a high level.
- [ ] Use `frozen=True` when an immutable-style model is appropriate.
- [ ] Explain what type hints do and do not guarantee.
- [ ] Use modern collection type annotations appropriately.
- [ ] Represent optional or union values clearly.
- [ ] Avoid unnecessary use of `Any`.
- [ ] Describe callable inputs with an appropriate callable type.
- [ ] Use `get_type_hints()` when introspection is actually needed.
- [ ] Separate external input validation from internal data modeling.
- [ ] Keep conversion to external schemas explicit.

---

## 26. Authoritative references

- Python `dataclasses` documentation: https://docs.python.org/3/library/dataclasses.html
- Python `typing` documentation: https://docs.python.org/3/library/typing.html
- Python tutorial — Classes: https://docs.python.org/3/tutorial/classes.html
- Python language reference — Compound statements / class definitions: https://docs.python.org/3/reference/compound_stmts.html#class-definitions
- Python standard library documentation: https://docs.python.org/3/library/

> **Source note:** The Python documentation is the source of truth for language and standard-library behavior. Version-specific features should be checked against the Python version targeted by the project.