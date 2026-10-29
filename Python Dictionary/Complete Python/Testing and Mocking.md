# Testing and Mocking

## Why this topic matters

DevOps automation is often responsible for actions that are easy to get wrong: changing configuration, building commands, parsing responses, calling APIs, and reacting to failures. Tests provide a repeatable way to verify behavior before automation is trusted with real systems.

Mocking is useful when a test needs to isolate the code under test from an external dependency such as a subprocess, filesystem boundary, network client, clock, or other object.

> **Verified fact:** Python's standard library provides the `unittest` framework and the `unittest.mock` module.
>
> **Repository design decision:** The examples in this guide emphasize automation code and dependency boundaries rather than a particular third-party test framework.

---

## 1. What a test should prove

A useful unit test normally establishes a relationship between an input and an expected result or observable behavior.

For example, if a function converts a command result into a status value, a test can verify that known inputs produce the expected status.

```python

def classify_exit_code(code: int) -> str:
    if code == 0:
        return "success"
    return "failure"
```

A focused test can verify the contract without running a real command:

```python
import unittest


class TestClassifyExitCode(unittest.TestCase):
    def test_success(self):
        self.assertEqual(classify_exit_code(0), "success")

    def test_failure(self):
        self.assertEqual(classify_exit_code(1), "failure")


if __name__ == "__main__":
    unittest.main()
```

The exact test runner used by a project can vary. What matters is that tests are deterministic, understandable, and tied to behavior that the project intends to preserve.

---

## 2. `unittest` mental model

The `unittest` framework provides several building blocks:

- `TestCase` — a class containing related test methods.
- test methods — methods whose names normally begin with `test`.
- assertions — methods such as `assertEqual()` and `assertRaises()` that check expected behavior.
- fixtures — setup and cleanup mechanisms used to prepare or release test state.
- test suites and runners — mechanisms for grouping and executing tests.

A minimal test case looks like this:

```python
import unittest


class TestDeploymentName(unittest.TestCase):
    def test_name_is_preserved(self):
        name = "payments-api"
        self.assertEqual(name, "payments-api")
```

Keep each test focused enough that a failure points toward a small area of behavior.

---

## 3. Assertions

Common `TestCase` assertions include:

```python
self.assertEqual(actual, expected)
self.assertNotEqual(actual, expected)
self.assertTrue(value)
self.assertFalse(value)
self.assertIsNone(value)
self.assertIsNotNone(value)
self.assertIn(item, collection)
```

When testing failures, `assertRaises()` verifies that a callable raises the expected exception:

```python
with self.assertRaises(ValueError):
    parse_port("not-a-port")
```

The important principle is to test the behavior the caller depends on, rather than implementation details that can change without changing the contract.

---

## 4. Testing exceptions

Automation code should make failure behavior explicit.

Suppose a parser rejects an invalid value:

```python

def parse_port(value: str) -> int:
    port = int(value)
    if not 1 <= port <= 65535:
        raise ValueError("port must be between 1 and 65535")
    return port
```

A test can verify both the exception type and, when the message is part of the intended contract, the message:

```python
class TestParsePort(unittest.TestCase):
    def test_invalid_port(self):
        with self.assertRaises(ValueError):
            parse_port("70000")
```

Avoid testing an exception merely because the current implementation happens to raise one. Test it when the failure behavior is meaningful to callers.

---

## 5. Test fixtures

Fixtures prepare state before a test and clean it up afterward.

`setUp()` runs before each test method in a `TestCase`, while `tearDown()` is used for cleanup after a test.

```python
class TestConfig(unittest.TestCase):
    def setUp(self):
        self.config = {"environment": "test"}

    def tearDown(self):
        self.config.clear()

    def test_environment(self):
        self.assertEqual(self.config["environment"], "test")
```

For resources that must be cleaned up even when setup or a test fails, use the cleanup mechanisms provided by `unittest` or a context manager appropriate to the resource.

A good fixture should be small. Large shared fixtures can make tests depend on one another and make failures harder to diagnose.

---

## 6. Testing filesystem and command boundaries

A common DevOps design is to keep external operations at a boundary and keep decision logic separate.

For example:

```python
import subprocess


def run_health_check(command: list[str]) -> bool:
    result = subprocess.run(command, check=False)
    return result.returncode == 0
```

The decision logic is simple, but a unit test should not need to execute the real command. This is where mocking can isolate the subprocess boundary.

---

## 7. What is a mock?

`unittest.mock` provides objects that can stand in for parts of a system during a test.

A mock can allow a test to:

- control what a dependency returns;
- verify that a dependency was called;
- inspect arguments passed to a dependency;
- simulate an exception;
- isolate the unit under test from an external system.

The goal is not to make every test use mocks. The goal is to isolate a dependency when doing so produces a clearer, faster, more deterministic unit test.

---

## 8. `Mock`

A basic mock can be configured with a return value:

```python
from unittest.mock import Mock

client = Mock()
client.get_status.return_value = "healthy"

assert client.get_status() == "healthy"
```

A test can also inspect how the mock was used:

```python
client.get_status.assert_called_once_with()
```

This is useful when the interaction with a dependency is part of the behavior being tested.

---

## 9. Patching a dependency

`patch()` temporarily replaces a target with a mock for the duration of a test or context.

```python
from unittest.mock import patch


@patch("deployment.subprocess.run")
def test_health_check(mock_run):
    mock_run.return_value.returncode = 0

    assert run_health_check(["health-check"]) is True
    mock_run.assert_called_once_with(["health-check"], check=False)
```

### The important rule: patch where the dependency is looked up

If a module imports a name into its own namespace, patch the name used by that module, not automatically the original definition elsewhere.

For example, if `deployment.py` contains:

```python
from subprocess import run
```

then code in `deployment.py` looks up `run` in the `deployment` module namespace. A test should therefore patch the lookup location used by that code.

This is one of the most important mocking concepts to understand because patching the wrong location can leave the real dependency active.

---

## 10. `return_value` and `side_effect`

`return_value` controls what a mock returns when called:

```python
mock_client.get.return_value = {"status": "ok"}
```

`side_effect` can be used for behavior such as raising an exception:

```python
mock_client.get.side_effect = TimeoutError("request timed out")
```

It can also be used with an iterable to provide successive results, or with a callable to calculate a result from the call arguments.

Use these features to model meaningful dependency behavior, not to recreate an entire external system inside the test.

---

## 11. `MagicMock`

`MagicMock` is a mock class that supports many magic methods and is useful when the object under test uses Python protocols such as iteration, item access, or context-manager behavior.

For example, a dependency used as a context manager can be represented by a mock configured for that protocol.

Use `Mock` when ordinary method calls are enough. Use `MagicMock` when the test specifically needs supported magic-method behavior.

---

## 12. `spec` and `autospec`

Mocks can accidentally hide interface mistakes because a basic mock can accept attributes that the real object does not provide.

`spec` can constrain a mock based on another object or type. `autospec` can create a mock using the target's interface and call signature.

This can make tests more resistant to accidental API drift.

Example:

```python
from unittest.mock import create_autospec


class Client:
    def get_status(self, service: str) -> str:
        raise NotImplementedError


client = create_autospec(Client)
client.get_status.return_value = "healthy"
```

The exact choice between unrestricted mocks, `spec`, and autospeccing is a repository design decision. Prefer stronger interface checking when it improves confidence without making tests unnecessarily difficult to maintain.

---

## 13. Testing call behavior

Mocks can verify calls:

```python
mock_client.restart.assert_called_once_with("payments-api")
```

They can also expose call information such as `call_count` and `call_args`.

Do not assert every internal call merely because the mock makes it possible. Excessive interaction assertions couple tests to implementation details and can make harmless refactoring painful.

A useful question is:

> Is this interaction part of the behavior that the component promises?

If yes, verify it. If no, prefer asserting the observable result.

---

## 14. Mocking versus integration testing

A unit test with mocks answers questions such as:

- Did this function make the expected decision?
- Did it handle a dependency failure correctly?
- Did it pass the right arguments to a dependency?

An integration test answers different questions:

- Do the real components work together?
- Does the actual API/client/database interaction behave as expected?
- Does the deployed configuration work across the boundary?

Mocks cannot prove that a real external system behaves correctly. For important integrations, use appropriate integration or end-to-end tests in addition to unit tests.

> **Repository design decision:** This repository treats unit tests and integration tests as complementary rather than interchangeable.

---

## 15. Testing command execution safely

For command-execution automation, avoid tests that accidentally modify the developer's machine or a real environment.

A good design separates command construction from command execution:

```python

def build_restart_command(service: str) -> list[str]:
    return ["systemctl", "restart", service]
```

The command construction can be tested without executing anything:

```python
class TestCommand(unittest.TestCase):
    def test_restart_command(self):
        self.assertEqual(
            build_restart_command("nginx"),
            ["systemctl", "restart", "nginx"],
        )
```

The execution boundary can then be tested separately with a mock.

This separation makes destructive operations easier to reason about and reduces the chance that a unit test accidentally touches a real system.

---

## 16. Testing retries and failures

Retry logic deserves explicit tests because it can otherwise hide subtle failure behavior.

A test should consider cases such as:

1. the first attempt succeeds;
2. an attempt fails and a later attempt succeeds;
3. every attempt fails;
4. the retry limit is reached;
5. the error should not be retried.

When time delays are involved, avoid making unit tests wait for real backoff intervals. Design the code so that the waiting boundary can be isolated or replaced in tests.

Do not use a mock to make an unsafe retry policy appear correct. The test should preserve the actual policy being evaluated.

---

## 17. Testing environment variables and configuration

Configuration-dependent code should be tested with explicit inputs rather than depending on whatever happens to be present in the developer's environment.

For example, if a function reads an environment variable, the test can control the environment boundary and verify behavior for:

- the variable being present;
- the variable being absent;
- an invalid value;
- a safe default when a default is intentionally part of the design.

Keep secrets out of test source code and test output. Use non-sensitive test values when demonstrating configuration behavior.

---

## 18. Testing logging

Logging is usually an observable side effect rather than the primary result of a function.

Test log output when the log itself is part of an operational contract—for example, when an important event must be emitted at a particular level or with a particular structured field.

Do not make every unit test depend on exact human-readable log text unless that text is intentionally part of the interface.

Most importantly, tests should not contain real credentials, tokens, private keys, or production configuration.

---

## 19. Test naming and structure

A good test name should communicate the behavior being checked.

Prefer:

```text
returns_failure_when_command_exits_nonzero
```

over:

```text
test_case_2
```

A useful structure is:

```text
Arrange
Act
Assert
```

For example:

```python

def test_classifies_nonzero_exit_code():
    # Arrange
    code = 1

    # Act
    result = classify_exit_code(code)

    # Assert
    assert result == "failure"
```

The exact assertion style depends on the test framework. The structure is a reasoning pattern rather than a framework requirement.

---

## 20. Common mocking mistakes

### Mocking the wrong lookup location

Patching the original library instead of the name used by the module under test may leave the real dependency active.

### Mocking too much

If every internal object is mocked, a test can pass while the actual components do not work together.

### Asserting implementation details

Tests that assert every private call can break during harmless refactoring.

### Ignoring real integrations

A fully mocked API client test does not prove that the real API request works.

### Using real destructive operations

Unit tests should not restart production services, delete real files, modify real infrastructure, or perform other unintended side effects.

### Making tests depend on machine state

Tests should not require arbitrary local environment variables, credentials, paths, or installed commands unless the test is explicitly an integration/environment test.

### Hiding failures

A test should fail when the behavior under test is wrong. Avoid broad exception handling that converts a failure into a passing test.

---

## 21. A practical DevOps testing layout

A small automation project can separate production code from tests:

```text
project/
├── src/
│   └── automation/
│       ├── __init__.py
│       ├── commands.py
│       ├── config.py
│       └── health.py
└── tests/
    ├── test_commands.py
    ├── test_config.py
    └── test_health.py
```

> **Repository design decision:** This is an illustrative layout, not a requirement imposed by Python.

As a project grows, the repository can additionally distinguish unit tests from tests that require real external services or infrastructure.

---

## 22. Exercises

### Exercise 1 — Exit codes

Write tests for a function that maps exit codes to success/failure states.

Include zero, a nonzero code, and at least one edge case relevant to your design.

### Exercise 2 — Command construction

Create a function that builds a safe command argument list for a service operation. Test the returned list without executing the command.

### Exercise 3 — Mock subprocess execution

Create a function that runs a command and returns a boolean based on its exit code. Mock the subprocess boundary and test both success and failure.

### Exercise 4 — API failure

Create a small client wrapper and test how your code handles a timeout or other expected dependency failure using `side_effect`.

### Exercise 5 — Retry behavior

Write a function with bounded retries and test success on the first attempt, eventual success, and exhaustion of retries.

### Exercise 6 — Configuration

Test configuration loading when a required environment variable is present, missing, and invalid.

### Exercise 7 — Integration boundary

Identify one component in a DevOps project that should be tested with a real integration test rather than only mocks. Explain why a mock cannot establish the same confidence.

---

## 23. Knowledge checks

Before moving on, you should be able to answer:

1. What is the purpose of `unittest.TestCase`?
2. When should `assertRaises()` be used?
3. What is the difference between `return_value` and `side_effect` on a mock?
4. Why does patching the lookup location matter?
5. When is `MagicMock` useful?
6. What problem do `spec` and `autospec` help address?
7. Why can excessive mocking reduce test confidence?
8. What can a unit test with mocks prove that an integration test proves differently?
9. How would you test command construction without executing the command?
10. Why should tests avoid production credentials and destructive operations?

---

## 24. Completion checklist

You are ready to move on when you can:

- write focused `unittest` test cases;
- choose assertions that describe intended behavior;
- test expected exceptions;
- use setup and cleanup appropriately;
- create and configure mocks;
- use `patch()` at the correct lookup location;
- distinguish `return_value` from `side_effect`;
- verify meaningful interactions without over-specifying implementation details;
- use `spec` or autospeccing when stronger interface checking is useful;
- separate unit tests from integration tests;
- test command, API, configuration, and retry boundaries safely;
- recognize when a mock is hiding a real integration problem.

---

## Authoritative references

- Python `unittest`: https://docs.python.org/3/library/unittest.html
- Python `unittest.mock`: https://docs.python.org/3/library/unittest.mock.html
- Python standard library: https://docs.python.org/3/library/

> **Source note:** The technical behavior described above is based on the official Python documentation for `unittest` and `unittest.mock`. Repository-specific layouts, exercises, and DevOps examples are instructional design choices for this repository.