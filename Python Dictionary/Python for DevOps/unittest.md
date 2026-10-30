# `unittest`

Testing is a core part of reliable DevOps automation. Scripts that modify files, call APIs, execute commands, or change infrastructure should be testable without requiring every test run to touch a real production system.

> **Source note:** Python-specific behavior in this guide is based on the official Python documentation for `unittest` and `unittest.mock`. Test architecture and DevOps examples are repository design decisions unless explicitly stated otherwise.

---

## 1. Testing mental model

A useful automation boundary is:

```text
input
  ↓
validation
  ↓
decision
  ↓
action
  ↓
result
```

Tests should verify the important behavior at these boundaries.

For example, a deployment helper might need to prove that:

- valid configuration produces the expected command;
- invalid configuration is rejected;
- a failed command is handled correctly;
- an API client receives the expected request;
- sensitive values are not accidentally exposed.

The goal is not to test every line simply because it exists. The goal is to create confidence in behavior that matters.

---

## 2. What `unittest` provides

Python's standard library includes the `unittest` framework.

It provides concepts including:

- **test case** — an individual unit of testing;
- **test fixture** — preparation and cleanup needed by tests;
- **test suite** — a collection of tests;
- **test runner** — the component that executes tests and reports results.

The central class for ordinary tests is `unittest.TestCase`.

Official source:

https://docs.python.org/3/library/unittest.html

---

## 3. A minimal test

```python
import unittest


def normalize_environment(value):
    return value.strip().lower()


class NormalizeEnvironmentTests(unittest.TestCase):
    def test_normalizes_value(self):
        self.assertEqual(normalize_environment(" PROD "), "prod")


if __name__ == "__main__":
    unittest.main()
```

A test method normally starts with `test` so that the test runner recognizes it as a test method.

The assertion expresses the expected behavior:

```python
self.assertEqual(actual, expected)
```

---

## 4. Common assertions

`TestCase` provides assertion methods designed to report useful test failures.

Common examples include:

```python
self.assertEqual(actual, expected)
self.assertNotEqual(actual, expected)
self.assertTrue(condition)
self.assertFalse(condition)
self.assertIsNone(value)
self.assertIsNotNone(value)
self.assertIn(item, collection)
self.assertNotIn(item, collection)
```

For exception behavior:

```python
with self.assertRaises(ValueError):
    parse_port("not-a-port")
```

Prefer the assertion that expresses the behavior most clearly.

---

## 5. Testing exceptions

Exception behavior is part of an automation interface.

For example:

```python

def parse_port(value):
    port = int(value)
    if not 1 <= port <= 65535:
        raise ValueError("port must be between 1 and 65535")
    return port
```

The test can verify the contract:

```python
class ParsePortTests(unittest.TestCase):
    def test_rejects_invalid_port(self):
        with self.assertRaises(ValueError):
            parse_port("70000")
```

If the exception message is part of a deliberate interface, it can also be checked. Otherwise, avoid making tests unnecessarily dependent on wording.

---

## 6. `setUp()` and `tearDown()`

Repeated preparation can be placed in `setUp()`:

```python
class ConfigTests(unittest.TestCase):
    def setUp(self):
        self.config = {
            "environment": "dev",
            "region": "example-region",
        }

    def test_environment(self):
        self.assertEqual(self.config["environment"], "dev")
```

`tearDown()` can perform cleanup after a test:

```python
class ResourceTests(unittest.TestCase):
    def setUp(self):
        self.resource = create_resource_for_test()

    def tearDown(self):
        self.resource.close()
```

The official documentation states that `setUp()` is called before each test method and `tearDown()` is run after a successful `setUp()`, whether the test itself succeeds or fails.

For resource cleanup, `addCleanup()` is another useful mechanism:

```python
self.addCleanup(resource.close)
```

This keeps cleanup close to the resource acquisition.

---

## 7. Keep tests independent

A test case should be able to run independently of other test cases.

Avoid designs such as:

```text
Test A creates shared state
        ↓
Test B expects that state
        ↓
Test C modifies it
```

Prefer:

```text
Test A → its own setup
Test B → its own setup
Test C → its own setup
```

This makes failures easier to reproduce and reduces order-dependent behavior.

The `unittest` documentation explicitly describes test cases as self-contained and suitable for execution in isolation or arbitrary combinations.

---

## 8. Test the decision, not the implementation detail

Suppose an automation function chooses whether a deployment should proceed:

```python

def should_deploy(environment, approved):
    return environment == "production" and approved
```

Test the behavior:

```python
class DeploymentDecisionTests(unittest.TestCase):
    def test_production_requires_approval(self):
        self.assertFalse(should_deploy("production", False))
        self.assertTrue(should_deploy("production", True))

    def test_non_production_does_not_deploy(self):
        self.assertFalse(should_deploy("staging", True))
```

The tests describe the policy rather than depending on internal implementation details.

---

## 9. Testing filesystem automation

Filesystem code should be designed so that tests can use temporary resources instead of real production paths.

A simple pattern is to make the path an input:

```python
from pathlib import Path


def read_config(path):
    return Path(path).read_text(encoding="utf-8")
```

The test can create a temporary file and verify the behavior without relying on a fixed machine-specific path.

For more involved temporary-resource tests, use the standard-library temporary-file facilities documented for Python rather than inventing fixed paths in the repository.

---

## 10. Testing environment-dependent code

Environment variables are an external boundary.

Instead of making every function read the environment directly, consider passing resolved configuration into the function:

```python

def build_endpoint(host, port):
    return f"https://{host}:{port}"
```

Then the environment-reading boundary can be tested separately.

When environment mutation is required in a test, `unittest.mock.patch.dict()` can temporarily modify a dictionary and restore it when the scope ends.

Example:

```python
from unittest.mock import patch

with patch.dict("os.environ", {"APP_ENV": "test"}):
    ...
```

The exact target and values are examples; choose them according to the application under test.

---

## 11. Why mocking matters in DevOps

DevOps automation frequently depends on external systems:

```text
Python code
   ↓
filesystem
   ↓
subprocess
   ↓
HTTP API
   ↓
cloud platform
   ↓
Kubernetes
```

A unit test should not need to invoke every external system to verify local decision logic.

`unittest.mock` provides mock objects that can replace parts of the system under test and record how they were used.

Official source:

https://docs.python.org/3/library/unittest.mock.html

---

## 12. Basic `Mock`

```python
from unittest.mock import Mock

client = Mock()
client.get_status.return_value = "healthy"

result = client.get_status()

self.assertEqual(result, "healthy")
client.get_status.assert_called_once_with()
```

A mock can provide a controlled return value and record calls made against it.

This is useful when the real dependency is slow, unavailable, expensive, destructive, or otherwise unsuitable for a unit test.

---

## 13. `return_value`

Use `return_value` when the dependency should return a known value:

```python
api = Mock()
api.get_status.return_value = {"status": "ready"}
```

Then the code under test can consume that response without making a real API call.

---

## 14. `side_effect`

`side_effect` can model failures or varying responses.

For example:

```python
api = Mock()
api.get_status.side_effect = TimeoutError("request timed out")
```

This allows a test to verify timeout handling without waiting for a real timeout.

It can also provide a sequence of results:

```python
api.get_status.side_effect = [
    {"status": "starting"},
    {"status": "ready"},
]
```

The official `unittest.mock` documentation supports exceptions, callables, and iterables as `side_effect` values.

---

## 15. Verify interactions

Mocks can verify how a dependency was called:

```python
client.deploy("service-a")

client.deploy.assert_called_once_with("service-a")
```

Other useful assertions include:

```python
mock.assert_called()
mock.assert_not_called()
mock.assert_called_once()
mock.assert_called_with(...)
mock.assert_called_once_with(...)
mock.assert_any_call(...)
```

Use interaction assertions when the interaction itself is important to the behavior being tested.

Do not turn every internal function call into a required interaction assertion. Excessive interaction testing can make refactoring unnecessarily difficult.

---

## 16. `patch()`

`patch()` temporarily replaces an object during a test and restores it afterward.

For example:

```python
from unittest.mock import patch

with patch("module_under_test.client") as client:
    client.get_status.return_value = "ready"
    ...
```

A critical rule from the official documentation is:

> Patch objects where they are looked up, not necessarily where they were originally defined.

Consider:

```python
# module_under_test.py
from api_client import Client
```

If the code under test uses `Client` from `module_under_test`, patching `api_client.Client` may not affect the already-imported reference. The patch target should correspond to the namespace used by the code under test.

---

## 17. `patch()` as a decorator

The same mechanism can be used as a decorator:

```python
@patch("module_under_test.Client")
def test_client(mock_client):
    ...
```

It can also be used as a context manager:

```python
with patch("module_under_test.Client") as mock_client:
    ...
```

Choose the form that makes the scope of the replacement obvious.

---

## 18. `MagicMock`

`MagicMock` is a `Mock` variant with support for many Python magic methods.

It can be useful when the code under test interacts with objects through protocols such as context management or iteration.

Do not choose `MagicMock` automatically. Use the simplest test double that represents the dependency boundary you need to control.

---

## 19. `spec` and `autospec`

Mocks can accidentally permit interactions that the real object does not support.

A `spec` restricts the available interface based on another object.

For example:

```python
mock_client = Mock(spec=RealClient)
```

Auto-speccing can also be used through `patch(..., autospec=True)` or `create_autospec()`.

The official documentation explains that auto-speccing can preserve attributes and call signatures from the object being replaced.

This can make tests catch incorrect calls that a completely unconstrained mock might otherwise accept.

---

## 20. Testing subprocess boundaries

A command-running function should be separated from the business decision that determines which command to execute.

For example, the decision layer can produce:

```python
command = ["tool", "deploy", "service-a"]
```

The execution layer can then invoke `subprocess.run()`.

Tests for the decision layer do not need to execute the real command.

Tests around the execution boundary can use a mock to verify:

- the executable and arguments;
- timeout configuration;
- whether failures are checked;
- how output is handled.

The exact command must be appropriate to the real tool being integrated; never invent a tool's CLI syntax.

---

## 21. Testing HTTP/API boundaries

A useful API client structure is:

```text
configuration
    ↓
authentication
    ↓
request construction
    ↓
HTTP boundary
    ↓
response parsing
    ↓
application decision
```

Unit tests can isolate the HTTP boundary and provide controlled responses.

For example, a mock HTTP client can simulate:

- success;
- authentication failure;
- authorization failure;
- timeout;
- malformed response;
- server failure.

This lets the application logic be tested without depending on the availability of a live service.

---

## 22. Testing retries

Retry logic is especially suitable for controlled tests.

A mock can produce a sequence such as:

```text
attempt 1 → timeout
attempt 2 → timeout
attempt 3 → success
```

Then the test can verify both:

1. the final result;
2. the number and nature of attempts.

Be careful not to make tests wait for real retry delays. Injecting a sleep function or mocking the delay boundary can keep tests deterministic.

Whether a particular failure should be retried is an application and protocol decision, not a universal Python rule.

---

## 23. Testing logging

Logging is another observable boundary.

Tests may need to verify that important failures are recorded without asserting every formatting detail.

More importantly for DevOps automation, tests should guard against accidental secret exposure.

For example, if a function handles an API token, a test can inspect the generated diagnostic information and verify that the token is not present.

The exact logging-test technique should match the application's logging architecture.

---

## 24. Unit tests versus integration tests

A practical distinction is:

### Unit test

Tests a small unit with external dependencies isolated or replaced.

```text
function
  ↓
mocked dependency
```

### Integration test

Tests how multiple real components interact.

```text
application
  ↓
real dependency
  ↓
real response
```

Both are useful.

For DevOps tooling, integration tests may be appropriate for carefully controlled environments such as:

- temporary directories;
- disposable databases;
- test APIs;
- sandbox cloud resources;
- dedicated Kubernetes test environments.

Do not accidentally turn ordinary unit tests into destructive integration tests.

---

## 25. Test discovery

Python provides test discovery through `unittest`.

A common command is:

```bash
python -m unittest discover
```

The discovery command supports options for the start directory, filename pattern, and top-level directory.

For example:

```bash
python -m unittest discover -s tests -p "test*.py"
```

These options are documented by Python and should be adjusted to match the repository's actual structure.

---

## 26. Running specific tests

You can run a module:

```bash
python -m unittest test_module
```

A test class:

```bash
python -m unittest test_module.TestClass
```

Or an individual test method:

```bash
python -m unittest test_module.TestClass.test_method
```

Verbose output can be requested with:

```bash
python -m unittest -v test_module
```

These are standard `unittest` command-line interfaces documented by Python.

---

## 27. Test selection with `-k`

Current Python documentation supports `-k` for selecting test methods and classes whose names match a pattern or substring.

For example:

```bash
python -m unittest -k deployment
```

Use this when narrowing a larger test run during development.

When documenting commands for a repository, verify the Python version used by the repository because command-line features can vary between Python releases.

---

## 28. Subtests

`subTest()` is useful when several closely related cases share the same test structure.

For example:

```python
class EnvironmentTests(unittest.TestCase):
    def test_normalization(self):
        cases = {
            " DEV ": "dev",
            "Prod": "prod",
            "STAGING": "staging",
        }

        for raw, expected in cases.items():
            with self.subTest(raw=raw):
                self.assertEqual(normalize_environment(raw), expected)
```

A failing subtest identifies the particular case without requiring separate test methods for every small variation.

---

## 29. Skipping tests

Sometimes a test depends on a platform or optional resource.

`unittest` supports decorators such as:

```python
@unittest.skip("requires external system")
```

and conditional forms such as `skipIf()` and `skipUnless()`.

A test can also call:

```python
self.skipTest("resource unavailable")
```

Skipping should describe a genuine test prerequisite, not hide a failing test.

---

## 30. Testing configuration validation

Configuration validation is a strong DevOps testing target.

For example:

```python

def validate_config(config):
    if "environment" not in config:
        raise ValueError("environment is required")
    if "region" not in config:
        raise ValueError("region is required")
```

Tests should cover:

```text
valid configuration
missing required field
invalid value
unexpected value
boundary value
```

This creates a clear contract before configuration reaches an external system.

---

## 31. Testing exit behavior

CLI automation often needs meaningful process outcomes.

Separate the core logic from the CLI boundary where possible:

```python

def run_job(config):
    ...
    return result


def main():
    result = run_job(load_config())
    return 0 if result.success else 1
```

The core function can then be tested independently of process termination.

If the CLI itself calls `sys.exit()`, test the boundary deliberately rather than allowing process termination to interfere with the rest of the test suite.

---

## 32. Tests should not contain real secrets

Never place production credentials in:

- test source;
- fixture files;
- sample JSON;
- committed environment files;
- expected-output files;
- mock configuration.

Use clearly fake values:

```python
TEST_TOKEN = "test-token"
```

Also test that sensitive values are redacted from logs and errors where appropriate.

---

## 33. Common testing mistakes

### Mistake 1: testing only the happy path

Include important failures and boundary conditions.

### Mistake 2: relying on test order

Tests should be independent.

### Mistake 3: hitting production systems in unit tests

Isolate external dependencies.

### Mistake 4: mocking everything

Keep important integration behavior covered by appropriate integration tests.

### Mistake 5: patching the wrong namespace

Patch where the dependency is looked up.

### Mistake 6: asserting implementation details everywhere

Prefer behavior-focused assertions.

### Mistake 7: using real credentials

Use fake values or controlled test credentials.

### Mistake 8: waiting through real timeouts and retry delays

Control time-dependent boundaries.

### Mistake 9: letting test fixtures leak state

Clean up resources and isolate test data.

### Mistake 10: hiding failures with excessive skips

Skip only when a documented prerequisite genuinely does not exist.

---

## 34. Practical exercise

Build a small deployment decision module with these functions:

```text
load configuration
        ↓
validate configuration
        ↓
determine deployment action
        ↓
invoke external deployment client
        ↓
return success/failure
```

Write `unittest` coverage for:

1. valid configuration;
2. missing configuration;
3. invalid configuration;
4. deployment approval logic;
5. successful deployment;
6. deployment failure;
7. timeout behavior;
8. verification that the deployment client received the expected arguments;
9. secret-safe diagnostics;
10. a retry sequence if your design includes retries.

Use mocks for the external deployment client.

Do not connect the exercise to a real production environment.

---

## 35. Knowledge checks

You should be able to answer:

1. What is a `TestCase`?
2. What is a test fixture?
3. Why should tests be independent?
4. When should `setUp()` and `tearDown()` be used?
5. When should `assertRaises()` be used?
6. What problem does `Mock` solve?
7. What is the difference between `return_value` and `side_effect`?
8. Why does the location patched by `patch()` matter?
9. What does `autospec` help protect against?
10. When should a unit test be replaced or supplemented by an integration test?
11. What does `subTest()` provide?
12. When is skipping a test appropriate?
13. How does `python -m unittest discover` help in CI?
14. Why should production credentials never be committed to test fixtures?
15. Why should retry tests avoid real delays?

---

## 36. Completion criteria

Consider this topic complete when you can:

- create a `unittest.TestCase`;
- write meaningful assertions;
- test expected exceptions;
- use fixtures and cleanup safely;
- keep tests independent;
- test configuration and filesystem boundaries;
- use `Mock`, `patch()`, and `side_effect` appropriately;
- understand where to patch dependencies;
- use `spec` or `autospec` when appropriate;
- test subprocess and API boundaries without invoking real systems;
- distinguish unit tests from integration tests;
- run tests with the `unittest` CLI and discovery;
- write tests that are safe for CI environments.

---

## 37. Verified facts vs repository design

### Verified facts

- `unittest.TestCase` is the standard base class for ordinary `unittest` test cases.
- Test methods are identified by the `test` naming convention.
- `setUp()` and `tearDown()` provide per-test setup and cleanup.
- `unittest` provides assertion methods such as `assertEqual()`, `assertTrue()`, and `assertRaises()`.
- `python -m unittest` provides a command-line interface for running tests.
- `unittest` provides test discovery.
- `unittest.mock` provides `Mock`, `MagicMock`, `patch()`, and related facilities.
- `side_effect` can model exceptions, callable behavior, or sequences of results.
- `patch()` temporarily replaces objects and restores them after the patch scope ends.
- Python's documentation states that objects should be patched where they are looked up.
- `autospec` and `create_autospec()` can constrain mocks according to real object interfaces and signatures.
- `subTest()` supports multiple related test cases within a test method.
- `unittest` supports skipping tests and expected failures.

### Repository design decisions

- DevOps examples emphasize configuration, subprocess, filesystem, HTTP, authentication, and deployment boundaries.
- Unit tests in this learning path should avoid real production systems.
- Fake credentials should be used in tests.
- Integration testing should be used for behavior that genuinely requires real external components.
- Tests should focus primarily on observable behavior rather than incidental implementation details.

---

## 38. Authoritative references

- Python `unittest`: https://docs.python.org/3/library/unittest.html
- Python `unittest.mock`: https://docs.python.org/3/library/unittest.mock.html
- Python testing documentation: https://docs.python.org/3/library/unittest.html#organizing-test-code

> **Source boundary:** Commands, APIs, and behaviors documented here are based on Python's official documentation. Authentication services, cloud APIs, Kubernetes APIs, deployment tools, and other external systems must be tested against their own authoritative documentation.
