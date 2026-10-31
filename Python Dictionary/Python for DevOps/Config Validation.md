# Config Validation

Configuration validation is the boundary between **parsing configuration** and **trusting configuration**.

A parser answers:

> “Can this input be decoded into Python values?”

Validation answers:

> “Does the decoded configuration satisfy the rules required by this program?”

This distinction is important in DevOps automation. A syntactically valid configuration can still contain a missing required key, an unsupported value, an invalid range, or a combination of settings that the application cannot safely use.

> **Source-first note:** Python's official documentation is the source of truth for the parsing APIs used here. The validation rules and examples are repository design demonstrations; Python's standard library does not provide one universal schema-validation framework for arbitrary configuration formats.

## 1. Configuration validation pipeline

A useful operational model is:

```text
configuration source
        ↓
      parse
        ↓
 structural validation
        ↓
 semantic validation
        ↓
 normalized configuration
        ↓
 application / automation
```

Keep these stages conceptually separate.

### Parsing

Convert a representation such as JSON, INI, or TOML into Python objects.

### Structural validation

Check that required sections, keys, and basic value types exist.

### Semantic validation

Check rules such as:

- a port is within an allowed range;
- a retry count is non-negative;
- an environment name belongs to an approved set;
- mutually dependent settings are present together.

### Normalization

Convert validated values into the representation the application expects.

## 2. Parsing is not validation

Consider JSON:

```python
import json

config = json.loads('{"port": 8080}')
```

The JSON parser can successfully produce a Python dictionary. That does not establish that `port` is a valid value for your application, nor that all required configuration is present.

Repository design decision:

> Treat successful parsing as a prerequisite for validation, not as proof that configuration is usable.

## 3. Required keys

For a small configuration object, explicit checks are often clearer than introducing a framework.

```python
REQUIRED_KEYS = {"environment", "port", "retries"}

missing = REQUIRED_KEYS - config.keys()
if missing:
    raise ValueError(f"Missing configuration keys: {sorted(missing)}")
```

This makes the contract visible in the code.

## 4. Validate types explicitly

Configuration data may originate as strings or other loosely typed values. Check the type your application actually requires.

```python
if not isinstance(config["port"], int):
    raise ValueError("port must be an integer")

if not isinstance(config["retries"], int):
    raise ValueError("retries must be an integer")
```

Be precise about booleans when integers are not acceptable. In Python, `bool` is a subclass of `int`, so this check:

```python
isinstance(True, int)
```

is true.

If the configuration contract requires an actual integer but not a boolean, use a stricter check:

```python
if type(config["retries"]) is not int:
    raise ValueError("retries must be an integer")
```

Use this deliberately; type validation should reflect the actual configuration contract.

## 5. Validate ranges

A value can have the correct type and still be invalid.

```python
port = config["port"]
if not 1 <= port <= 65535:
    raise ValueError("port must be between 1 and 65535")
```

Similarly:

```python
retries = config["retries"]
if retries < 0:
    raise ValueError("retries cannot be negative")
```

The exact permitted range is a **repository/application rule**, not a universal rule supplied by Python's configuration parsers.

## 6. Validate allowed values

For finite choices, an explicit set makes the contract easy to review.

```python
allowed_environments = {"dev", "staging", "production"}

environment = config["environment"]
if environment not in allowed_environments:
    raise ValueError("unsupported environment")
```

This pattern is useful for deployment modes, log levels, regions, strategies, and other enumerated settings.

## 7. Validate relationships between fields

Individual fields may be valid while their combination is invalid.

```python
if config["tls_enabled"] and not config["certificate_path"]:
    raise ValueError(
        "certificate_path is required when tls_enabled is true"
    )
```

These are semantic constraints. They belong after the configuration has been parsed and basic fields have been checked.

## 8. Build a dedicated validation function

Centralize validation rather than scattering checks throughout the application.

```python
def validate_config(config: dict) -> None:
    required = {"environment", "port", "retries"}
    missing = required - config.keys()
    if missing:
        raise ValueError(f"Missing keys: {sorted(missing)}")

    if type(config["port"]) is not int:
        raise ValueError("port must be an integer")
    if not 1 <= config["port"] <= 65535:
        raise ValueError("port is outside the allowed range")

    if type(config["retries"]) is not int:
        raise ValueError("retries must be an integer")
    if config["retries"] < 0:
        raise ValueError("retries cannot be negative")

    if config["environment"] not in {"dev", "staging", "production"}:
        raise ValueError("unsupported environment")
```

Then keep the application flow simple:

```python
config = load_config()
validate_config(config)
run_automation(config)
```

## 9. Return normalized configuration

For more complex applications, validation can also normalize values.

For example, a configuration contract might allow a value to be represented as a string and then convert it to the type used internally.

Keep this behavior explicit:

```python
def normalize_config(config: dict) -> dict:
    normalized = dict(config)
    normalized["environment"] = normalized["environment"].lower()
    return normalized
```

Do not silently normalize values when doing so could hide a configuration error. Decide which transformations are part of the configuration contract.

## 10. JSON configuration

The `json` module parses JSON into Python objects and raises `JSONDecodeError` when JSON cannot be decoded.

```python
import json

with open("config.json", encoding="utf-8") as f:
    config = json.load(f)

validate_config(config)
```

Python's JSON parser validates JSON syntax. Your application still needs to validate its own configuration contract.

The official documentation also exposes `python -m json` / `json.tool` for validating and pretty-printing JSON input.

## 11. INI configuration with `configparser`

`configparser` implements an INI-style configuration language.

```python
import configparser

parser = configparser.ConfigParser()
parser.read("config.ini")
```

Configuration parser values are fundamentally text values, so conversion and validation should be intentional.

For example, the parser supports typed accessors such as:

```python
port = parser["server"].getint("port")
enabled = parser["server"].getboolean("enabled")
```

These conversions help interpret values, but application-specific constraints still need to be checked.

## 12. TOML configuration

Python's `tomllib` provides TOML parsing in the standard library.

```python
import tomllib

with open("config.toml", "rb") as f:
    config = tomllib.load(f)

validate_config(config)
```

`tomllib` parses TOML and returns Python data structures. It does not define your application's configuration schema.

It is a parser, not a universal semantic validation framework.

## 13. Command-line validation with `argparse`

`argparse` can perform simple command-line type conversion and choices validation while parsing CLI arguments.

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--retries", type=int, default=3)
parser.add_argument(
    "--environment",
    choices=["dev", "staging", "production"],
)

args = parser.parse_args()
```

The official documentation recommends keeping more complex validation downstream after parsing rather than forcing complex resource or error handling into an `argparse` type converter.

This gives a useful boundary:

```text
CLI syntax/type conversion → argparse
application configuration rules → validation code
```

## 14. Dataclasses do not automatically validate values

A dataclass is useful for representing a validated configuration object:

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class AppConfig:
    environment: str
    port: int
    retries: int
```

However, Python's `@dataclass` machinery does not automatically enforce the types written in annotations.

Validation still needs to happen explicitly:

```python
def build_config(raw: dict) -> AppConfig:
    validate_config(raw)
    return AppConfig(**raw)
```

This separation makes the boundary clear:

```text
raw configuration
      ↓
validation
      ↓
AppConfig
      ↓
application
```

## 15. Fail early

Configuration errors should normally be detected before destructive or irreversible automation begins.

Prefer:

```text
load → parse → validate → log readiness → execute
```

over:

```text
load → start deployment → discover invalid setting halfway through
```

For infrastructure and deployment tooling, early failure reduces partial execution and makes troubleshooting easier.

## 16. Produce useful validation errors

A useful validation error identifies the failing field and the violated rule.

Less useful:

```text
invalid configuration
```

More useful:

```text
configuration field 'retries' must be >= 0; received -1
```

For multiple independent validation errors, a repository may choose to collect them and report them together rather than stopping at the first error.

That is a **repository design decision**, not a requirement of Python's standard library.

## 17. Do not expose secrets in errors

Configuration validation often handles credentials, tokens, private keys, and other sensitive values.

Do not include secret values in validation exceptions or logs.

Prefer:

```text
configuration field 'api_token' is missing
```

over:

```text
api_token=super-secret-value is invalid
```

The exact secret-management mechanism belongs to the surrounding system; validation code should nevertheless avoid unnecessarily copying secrets into diagnostics.

## 18. Test the validation boundary

Test valid and invalid configurations independently.

```python
import unittest

class TestConfigValidation(unittest.TestCase):
    def test_valid_config(self):
        config = {
            "environment": "production",
            "port": 443,
            "retries": 3,
        }
        validate_config(config)

    def test_negative_retries(self):
        config = {
            "environment": "production",
            "port": 443,
            "retries": -1,
        }
        with self.assertRaises(ValueError):
            validate_config(config)
```

High-value tests include:

- missing required fields;
- wrong types;
- boundary values;
- values just outside allowed ranges;
- unsupported enumerated values;
- invalid combinations of fields;
- malformed input documents;
- secret-safe error messages.

## 19. Common mistakes

### Mistake 1: Treating parsing as validation

A syntactically valid document can still violate application requirements.

### Mistake 2: Trusting type annotations as runtime validation

Annotations and dataclass fields do not automatically enforce configuration values.

### Mistake 3: Validating too late

Do not begin important operational work before configuration is known to be valid.

### Mistake 4: Scattering validation everywhere

Centralize configuration validation so the contract is easy to review and test.

### Mistake 5: Logging sensitive configuration

Never dump complete configuration objects merely to diagnose a validation failure when they may contain secrets.

### Mistake 6: Silently accepting unknown settings

Whether unknown keys should be rejected or ignored is an application design decision. Make it explicit instead of accidental.

### Mistake 7: Using complex `argparse` converters for application logic

The Python documentation recommends keeping more involved validation downstream after argument parsing.

## 20. Practical DevOps exercise

Build a configuration loader with this contract:

```text
environment: dev | staging | production
port: integer from 1 through 65535
retries: integer >= 0
tls_enabled: boolean
certificate_path: required when tls_enabled is true
```

Requirements:

1. Load the configuration from JSON.
2. Parse the JSON.
3. Validate required fields.
4. Validate types.
5. Validate ranges and allowed values.
6. Validate the TLS/certificate relationship.
7. Convert the validated dictionary into a configuration object.
8. Write unit tests for valid and invalid cases.
9. Ensure errors do not print secret values.

## 21. Knowledge checks

Before moving on, you should be able to answer:

1. What is the difference between parsing and validation?
2. Why can a successfully parsed JSON document still be invalid for an application?
3. How would you validate a required configuration key?
4. How would you enforce an allowed set of environment names?
5. Why should semantic validation happen before deployment actions?
6. Do dataclass annotations automatically enforce runtime configuration types?
7. When should complex CLI validation happen relative to `argparse` parsing?
8. Why should validation errors avoid printing secret values?
9. What does `tomllib` provide, and what does it not provide?
10. Which validation rules are application-specific rather than Python parser behavior?

## 22. Completion criteria

You are ready to continue when you can:

- distinguish parsing from validation;
- validate required fields and types;
- enforce ranges and enumerated values;
- validate relationships between settings;
- handle JSON, INI, and TOML parsing appropriately;
- use `argparse` for simple CLI validation;
- represent validated configuration cleanly;
- test both valid and invalid configurations;
- produce useful, secret-safe validation errors;
- explain which behavior comes from Python and which comes from your application contract.

## Verified facts vs repository design

### Verified facts

- Python's `json` module parses JSON and raises `JSONDecodeError` for invalid JSON documents.
- `configparser` provides INI-style configuration parsing and typed getter methods.
- `tomllib` parses TOML and returns Python objects; it does not provide a general application schema validator.
- `argparse` supports type conversion and choices for command-line arguments.
- Dataclass annotations describe fields but the `@dataclass` decorator does not generally enforce the annotated runtime types.

### Repository design decisions

- Validate configuration before operational execution.
- Keep validation in a dedicated boundary rather than scattering it through automation code.
- Treat secret-safe diagnostics as a repository quality requirement.
- Use explicit application rules for ranges, required fields, and cross-field relationships.

### Demonstrations

The configuration examples, validation functions, exercises, and test cases in this file are teaching examples created for this repository.

## Authoritative references

- Python `json`: https://docs.python.org/3/library/json.html
- Python `configparser`: https://docs.python.org/3/library/configparser.html
- Python `tomllib`: https://docs.python.org/3/library/tomllib.html
- Python `argparse`: https://docs.python.org/3/library/argparse.html
- Python `dataclasses`: https://docs.python.org/3/library/dataclasses.html
- Python standard library: https://docs.python.org/3/library/
