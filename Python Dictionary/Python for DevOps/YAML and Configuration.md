# YAML and Configuration

YAML is a human-readable data serialization format commonly used for configuration. Python's standard library does **not** include a YAML parser, so YAML support requires a separate package or tool. This topic focuses on configuration design and the boundary between configuration data and Python automation.

> **Source note:** Python-specific claims are based on the official Python documentation. YAML syntax and parser behavior must be verified against the documentation for the specific YAML implementation used by a project. The examples below are teaching demonstrations and do not assume a particular third-party YAML library unless stated.

---

## 1. The mental model

Treat configuration as external input:

```text
configuration source
        ↓
read
        ↓
parse
        ↓
validate
        ↓
normalize
        ↓
application logic
```

A configuration file should describe **what the program should do**, while application code should implement **how the operation is performed**.

This separation makes automation easier to review, test, and change.

---

## 2. Why YAML appears in DevOps

YAML is widely encountered in infrastructure and automation ecosystems. Typical configuration documents may describe:

- services
- environments
- deployment settings
- pipelines
- application configuration
- lists of hosts or targets
- feature settings

The exact YAML syntax, supported features, schema, and semantics depend on the tool consuming the YAML document.

### Important distinction

```text
YAML syntax
    ≠
tool-specific configuration schema
```

A syntactically valid YAML document can still be invalid for the application that consumes it.

---

## 3. Python does not provide a built-in YAML module

Python's standard library provides modules such as `json`, but YAML parsing is not part of the standard `json` or general standard-library interfaces.

Therefore, do not write:

```python
import yaml
```

and assume that it is available in a normal Python installation.

The `yaml` import convention is associated with third-party YAML implementations. The exact package, supported YAML version, and security properties must be verified before adding a dependency.

> **Repository rule:** Never introduce an unverified third-party dependency into an example merely because it is commonly used.

---

## 4. YAML versus JSON

Both YAML and JSON can represent structured data, but they have different syntax and ecosystem characteristics.

Conceptually:

```text
YAML
 ↓
structured data
 ↓
Python objects
```

and:

```text
JSON
 ↓
structured data
 ↓
Python objects
```

JSON support is available in Python's standard library through `json`. YAML requires an appropriate external implementation when Python needs to parse it directly.

For simple configuration, choose a format based on the requirements of the surrounding tool and project rather than assuming one format is universally better.

---

## 5. A simple YAML document

A generic YAML document might look like:

```yaml
service:
  name: api
  replicas: 3

environment: production
```

The important lesson is the **data structure** being represented:

```text
service
 ├── name
 └── replicas

environment
```

The consuming program is responsible for interpreting that structure according to its configuration contract.

---

## 6. Configuration is an API boundary

Treat configuration as an interface between humans, deployment systems, and application code.

A useful design is:

```text
configuration file
       ↓
configuration loader
       ↓
validated configuration
       ↓
application
```

Avoid allowing configuration parsing to leak throughout the application.

Instead, centralize it:

```python
def load_config(path):
    # read and parse configuration
    ...


def validate_config(config):
    # validate required fields and values
    ...
```

The exact parser implementation is intentionally omitted here because the repository does not assume a particular third-party YAML library.

---

## 7. Required and optional settings

Configuration should clearly distinguish required values from optional values.

For example:

```text
Required
────────
service.name
service.replicas

Optional
────────
service.timeout
service.region
```

A missing required value should normally produce a clear configuration error rather than silently selecting an unsafe or surprising value.

An optional value can have a documented default when that default is appropriate.

This is a **repository design principle**, not a rule imposed by YAML itself.

---

## 8. Validate structure after parsing

Parsing answers:

> "Can this document be interpreted as data?"

Validation answers:

> "Is this data acceptable for my application?"

These are different questions.

For example, a configuration may contain:

```yaml
replicas: three
```

The document can be syntactically valid YAML while still violating an application's requirement that `replicas` be an integer.

A useful boundary is:

```text
parse
  ↓
required fields
  ↓
type checks
  ↓
value constraints
  ↓
semantic checks
  ↓
use configuration
```

---

## 9. Normalize configuration

External configuration can contain representations that are convenient for humans but awkward for application code.

A configuration layer can normalize those values before application logic uses them.

For example:

```python
def normalize_config(config):
    return {
        "service_name": config["service"]["name"],
        "replicas": config["service"]["replicas"],
    }
```

The goal is to keep the rest of the application working with a stable internal representation.

---

## 10. Environment variables and configuration files

Environment variables and configuration files solve related but different problems.

A practical configuration model may be:

```text
configuration file
        ↓
base configuration
        ↓
environment variables
        ↓
explicit overrides
        ↓
final validated configuration
```

Whether environment variables override file values is an **application design decision**. Do not assume that every tool uses the same precedence rules.

If a repository demonstrates precedence, document that precedence explicitly.

---

## 11. Do not confuse configuration with secrets

A configuration file may contain ordinary settings, but it can also accidentally contain credentials.

For example, avoid committing values such as:

```yaml
username: example
password: real-secret
```

Use the secret-management mechanism appropriate to the platform instead.

Even when configuration is represented as YAML, the format itself provides no secret-management guarantee.

### Safe practice

```text
configuration
    ↓
non-sensitive settings

secret system / protected environment
    ↓
sensitive values
```

Do not print complete configuration objects to logs without considering whether they contain sensitive information.

---

## 12. YAML parser security

YAML implementations can provide features beyond simple data representation. Some implementations have historically supported object construction or other behaviors that can be unsafe when processing untrusted input.

Therefore:

- verify the parser's documentation
- understand which loading mode is intended for untrusted data
- keep dependencies current according to the project's maintenance policy
- do not blindly use a loader from an example found online

Because parser behavior is implementation-specific, this repository does not claim a universal safe loader name without verifying the selected YAML implementation's current documentation.

> **Security rule:** If the parser and loading mode have not been verified from authoritative documentation, this could not be verified from the available authoritative sources.

---

## 13. YAML anchors and advanced syntax

YAML supports features beyond simple mappings and lists, depending on the YAML specification and parser implementation.

Examples may include:

- anchors
- aliases
- multiline scalars
- tags
- flow-style collections

Do not assume that every tool accepts every YAML feature.

For DevOps work, always check the consuming tool's documentation before relying on advanced YAML syntax.

---

## 14. YAML schema versus YAML syntax

Consider a document:

```yaml
replicas: 3
```

YAML syntax determines how the document is represented as data.

A tool's configuration schema determines whether `replicas` is:

- allowed
- required
- an integer
- constrained to a range
- ignored
- invalid

Those rules belong to the consuming application's schema, not to generic YAML syntax.

This distinction is especially important when working with Kubernetes, CI/CD systems, or cloud tooling.

---

## 15. Configuration errors should fail early

A useful automation pattern is:

```text
start
 ↓
load configuration
 ↓
validate configuration
 ↓
if invalid → fail
 ↓
perform external actions
```

Do not begin destructive or externally visible operations before validating the configuration required to perform them safely.

For example, validate a target environment before starting a deployment operation.

This is a repository design principle intended to make automation safer.

---

## 16. Keep configuration parsing separate from actions

Prefer:

```python
def load_config(path):
    ...


def validate_config(config):
    ...


def deploy(config):
    ...
```

over a single function that simultaneously:

```text
read file
→ parse YAML
→ validate
→ call cloud API
→ modify infrastructure
```

Separating responsibilities makes the configuration layer easier to test without performing real infrastructure operations.

---

## 17. Configuration precedence must be explicit

If multiple configuration sources exist, define their precedence.

For example, a project might intentionally choose:

```text
built-in defaults
       ↓
configuration file
       ↓
environment variables
       ↓
CLI arguments
```

But another application may use a different order.

There is no universal precedence rule supplied by YAML itself.

Document the actual precedence chosen by the application.

---

## 18. Avoid implicit type assumptions

Configuration values should be validated explicitly.

For example, do not assume that a value is safe to use merely because it exists:

```python
replicas = config["replicas"]
```

Instead, validate the expected type and constraints before use.

Conceptually:

```python
replicas = config.get("replicas")

if not isinstance(replicas, int):
    raise ValueError("replicas must be an integer")

if replicas < 1:
    raise ValueError("replicas must be positive")
```

The exact validation rules belong to the application.

---

## 19. Testing configuration

Configuration handling should be tested separately from the external operation it controls.

Useful test cases include:

```text
valid configuration
missing required field
wrong type
invalid value
unknown field
empty document
malformed document
conflicting configuration sources
sensitive value handling
```

The exact behavior for unknown fields or empty documents must be defined by the application or consuming tool.

---

## 20. DevOps practice exercise

Design a configuration loader with this flow:

```text
config.yaml
    ↓
read file
    ↓
parse using a verified YAML implementation
    ↓
validate required fields
    ↓
validate types and ranges
    ↓
apply documented overrides
    ↓
return normalized configuration
```

Requirements:

1. Keep the YAML parser dependency explicit.
2. Do not assume a parser is part of Python's standard library.
3. Require a service name.
4. Require a valid replica count.
5. Separate parsing from validation.
6. Keep secrets out of the example configuration.
7. Do not perform external infrastructure actions during configuration loading.
8. Document configuration precedence.

---

## 21. Knowledge checks

Before moving on, you should be able to answer:

1. Is YAML part of Python's standard library?
2. What is the difference between parsing and validation?
3. Why can syntactically valid YAML still be invalid application configuration?
4. Why should the exact YAML parser be treated as a dependency decision?
5. Why should configuration loading be separated from infrastructure actions?
6. Why should secrets not be treated as ordinary YAML configuration?
7. What determines whether an advanced YAML feature is accepted by a DevOps tool?
8. Why must configuration precedence be documented?
9. Why should configuration errors be detected before external actions begin?
10. What information must be verified before choosing a YAML loader for untrusted input?

---

## 22. Completion criteria

You are ready to move on when you can:

- explain YAML's role as a configuration/data format
- distinguish YAML syntax from a tool's configuration schema
- explain why YAML parsing requires an appropriate implementation in Python
- separate parsing from validation
- validate required fields and value types
- design explicit configuration precedence
- distinguish configuration from secrets
- recognize parser security considerations
- normalize external configuration into an internal representation
- test configuration independently of external infrastructure actions

---

## 23. Source classification

### Verified facts

Python's standard library documentation establishes the standard-library boundary and provides the Python APIs used elsewhere in this learning path. YAML parser behavior is implementation-specific and should be verified against the selected parser's authoritative documentation.

### Repository design decisions

The `read → parse → validate → normalize → act` workflow, explicit configuration precedence, separation of configuration from actions, and secret-handling guidance are teaching and repository design choices.

### Demonstrations

The YAML snippets and Python validation examples are intentionally generic demonstrations. They are not claims about Kubernetes, GitHub Actions, Terraform, or any other specific configuration schema.

---

## 24. Authoritative references

- Python standard library: https://docs.python.org/3/library/
- Python `json` module: https://docs.python.org/3/library/json.html
- Python `os` module: https://docs.python.org/3/library/os.html
- Python `pathlib` module: https://docs.python.org/3/library/pathlib.html
- Python Packaging User Guide: https://packaging.python.org/en/latest/

For YAML itself, use the official documentation of the specific YAML implementation selected for the project. This repository intentionally does not invent a package name, version, loader API, or security guarantee without verifying that implementation's current authoritative documentation.