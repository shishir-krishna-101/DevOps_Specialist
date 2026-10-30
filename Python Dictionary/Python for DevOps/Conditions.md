# Conditions

Conditions are how Python automation turns observed values into decisions. DevOps scripts constantly need to answer questions such as whether a configuration is valid, whether a service is healthy, whether an environment is allowed, or whether an action should proceed.

The goal is to write conditions that are correct, explicit, and easy to review.

---

## 1. The basic `if` statement

Python uses `if` to execute a block when a condition is truthy.

```python
healthy = True

if healthy:
    print("Service is healthy")
```

The indented block belongs to the `if` statement.

A useful automation mental model is:

```text
observed state
     ↓
condition
     ↓
decision
     ↓
action
```

Keep the condition understandable enough that another engineer can verify the operational decision.

---

## 2. `if`, `elif`, and `else`

Use `elif` for additional mutually exclusive branches and `else` for the remaining case.

```python
status = "degraded"

if status == "healthy":
    print("Continue")
elif status == "degraded":
    print("Investigate")
else:
    print("Stop")
```

Python checks the branches in order. Once a matching condition is found, its block runs and later branches are skipped.

This makes ordered policy decisions easy to express.

---

## 3. Conditions evaluate truth values

A condition does not have to literally be `True` or `False`.

Python performs truth-value testing for objects used in conditional contexts.

```python
services = []

if not services:
    print("No services configured")
```

Common false values include `None`, `False`, numeric zero, and empty containers or strings.

Do not use truthiness when different false values have different operational meanings.

For example, these may represent different states:

```python
value = None
value = 0
value = ""
```

If those meanings matter, test explicitly.

---

## 4. Comparison conditions

Comparison operators are frequently used in automation decisions.

```python
current_replicas = 2
desired_replicas = 3

if current_replicas < desired_replicas:
    print("Scaling may be required")
```

Common comparisons include:

```text
<   <=   >   >=   ==   !=
```

The comparison produces a Boolean result that controls the branch.

---

## 5. Equality versus identity

Use `==` for value equality.

```python
status = "healthy"

if status == "healthy":
    print("Healthy")
```

Use `is` when object identity matters. The most common example is checking for `None`.

```python
result = None

if result is None:
    print("No result")
```

Do not use `is` as a general replacement for `==`.

---

## 6. Combining conditions with `and`

Use `and` when multiple conditions must be satisfied.

```python
environment = "production"
healthy = True

if environment == "production" and healthy:
    print("Production service is healthy")
```

Python evaluates `and` from left to right and short-circuits when the first operand is false.

This can be useful when the second condition depends on the first.

```python
config = None

if config is not None and config.get("enabled"):
    print("Enabled")
```

If `config is None`, the second expression is not evaluated.

---

## 7. Alternatives with `or`

Use `or` when at least one condition can satisfy the decision.

```python
environment = "staging"
maintenance_override = False

if environment == "production" or maintenance_override:
    print("Special policy applies")
```

Remember that Python's `or` returns one of its operands, not necessarily a Boolean.

This pattern can therefore be surprising:

```python
timeout = configured_timeout or 30
```

If `configured_timeout` is `0`, the fallback is selected because `0` is false.

When zero is a valid value, test for absence explicitly instead.

---

## 8. Negating a condition with `not`

Use `not` when the opposite truth value expresses the policy clearly.

```python
maintenance_mode = False

if not maintenance_mode:
    print("Normal operations")
```

Avoid chains of negations that make a condition difficult to understand. Rewrite the policy in positive terms when that improves clarity.

---

## 9. Membership conditions

Use `in` and `not in` for membership checks.

```python
allowed_environments = {"dev", "test", "production"}
environment = "production"

if environment in allowed_environments:
    print("Environment allowed")
```

For dictionaries, `in` tests keys by default.

```python
config = {"region": "ap-south-1"}

if "region" in config:
    print("Region configured")
```

If you need to inspect values or key/value pairs, use the appropriate dictionary operation explicitly.

---

## 10. Nested conditions

Conditions can contain other conditions.

```python
if environment == "production":
    if healthy:
        print("Production service is healthy")
```

Nested logic can be appropriate when the second decision genuinely depends on the first.

However, deeply nested conditions are harder to review and test. Look for opportunities to express simple policy directly.

For example:

```python
if environment == "production" and healthy:
    print("Production service is healthy")
```

Choose the form that makes the policy easiest to verify.

---

## 11. Guard clauses

A guard clause handles an invalid or disallowed state early.

```python
def deploy(environment, healthy):
    if environment not in {"staging", "production"}:
        raise ValueError("unsupported environment")

    if not healthy:
        raise RuntimeError("service is not healthy")

    print("Proceed with deployment")
```

Guard clauses can reduce nesting and make failure conditions explicit.

This is a repository design recommendation for readable automation, not a Python requirement.

---

## 12. Conditions and validation

Validation should happen before an operational action.

```python
port = 443

if not 1 <= port <= 65535:
    raise ValueError("invalid port")
```

A chained comparison expresses the range directly.

For configuration, distinguish validation from action:

```python
if environment not in {"dev", "test", "production"}:
    raise ValueError("unsupported environment")

# action happens only after validation
```

This separation makes the code easier to test and reason about.

---

## 13. Conditional expressions

A conditional expression selects one of two values.

```python
status = "healthy" if healthy else "unhealthy"
```

Use this for simple value selection.

Avoid turning substantial operational control flow into a single line.

Prefer normal `if` statements when the branches involve multiple operations, exceptions, logging, or other side effects.

---

## 14. Conditions with collections

Conditions often inspect collection state.

```python
failed_services = ["api", "worker"]

if failed_services:
    print("Failures detected")
```

Membership can be combined with collection state:

```python
critical_services = {"api", "database"}
changed_services = {"api", "frontend"}

if critical_services & changed_services:
    print("Critical service changed")
```

The exact collection operation should reflect the intended question. Do not rely on a vague truthiness check when you need to know which items caused the condition.

---

## 15. Conditions and configuration values

Configuration frequently arrives as text and must be converted before numeric or Boolean comparisons.

```python
raw_replicas = "3"
replicas = int(raw_replicas)

if replicas < 3:
    print("Capacity is below target")
```

For Boolean configuration, avoid assuming arbitrary strings behave like Boolean values.

For example, the string `"false"` is a non-empty string and is therefore truthy:

```python
value = "false"

if value:
    print("This branch runs")
```

Parse and validate configuration according to an explicit format rather than relying on generic truthiness.

---

## 16. Conditions and exceptions

Some failures are better represented by exceptions than by a Boolean condition.

For example, configuration validation can fail explicitly:

```python
if environment not in allowed_environments:
    raise ValueError("unsupported environment")
```

The caller can then decide how the failure should be handled.

Do not hide operational failures by converting every failure into `False` without preserving useful context.

---

## 17. Avoid side effects inside conditions

Conditions should primarily answer a question.

Prefer:

```python
if should_restart:
    restart_service()
```

over clever expressions whose main purpose is to trigger an action indirectly.

This keeps policy and mechanism separate:

```text
policy  → should we act?
mechanism → how do we act?
```

This separation is a repository design recommendation for maintainable DevOps automation.

---

## 18. Make operational policy readable

A large condition can hide the actual policy.

Instead of:

```python
if environment == "production" and healthy and replicas >= 3 and not maintenance_mode:
    deploy = True
```

name meaningful decisions:

```python
production = environment == "production"
sufficient_capacity = replicas >= 3
safe_to_deploy = healthy and not maintenance_mode

deploy = production and sufficient_capacity and safe_to_deploy
```

The second version exposes the policy components and gives them names that can be tested independently.

Again, this is a repository design choice, not a Python requirement.

---

## 19. Common mistakes

### Mistake 1: Using `=` where a comparison is required

```python
status == "healthy"
```

is a comparison. Assignment uses `=`.

### Mistake 2: Using `is` for ordinary value comparison

Use `==` for values and `is` for identity.

### Mistake 3: Treating every false value as the same state

`None`, `0`, `False`, `""`, and empty containers can have different meanings.

### Mistake 4: Treating `"false"` as Boolean false

A non-empty string is truthy.

### Mistake 5: Overusing nested `if` statements

Deep nesting can hide the main operational policy.

### Mistake 6: Hiding side effects in expressions

Keep actions explicit.

### Mistake 7: Building one giant condition

Split meaningful policy decisions into named expressions when that improves reviewability.

### Mistake 8: Validating after the action

Check assumptions before deployment, deletion, restart, or other operational side effects.

---

## 20. Practical DevOps exercises

### Exercise 1 — Environment validation

Accept an environment value and allow only `dev`, `test`, and `production`.

### Exercise 2 — Health gate

Proceed only when a service is healthy and not in maintenance mode.

### Exercise 3 — Capacity gate

Require the current replica count to be at least the configured minimum before continuing.

### Exercise 4 — Guard clauses

Write a function that rejects an unsupported environment and an unhealthy service before performing an action.

### Exercise 5 — Configuration parsing

Given a string such as `"false"`, implement explicit Boolean parsing rules rather than relying on truthiness.

### Exercise 6 — Deployment policy

Create named conditions for environment, health, capacity, and maintenance state. Combine them into a final deployment decision.

### Exercise 7 — Failure reporting

Design a validation function that identifies which configuration rule failed instead of returning only a generic `False`.

---

## 21. Knowledge check

You should be able to explain:

1. How `if`, `elif`, and `else` are evaluated.
2. What truth-value testing means in Python.
3. When to use `==` versus `is`.
4. How `and`, `or`, and `not` combine conditions.
5. Why `or` can produce an unexpected configuration fallback.
6. What `in` means for a dictionary.
7. Why `"false"` is truthy.
8. When a guard clause improves automation code.
9. Why validation should occur before operational side effects.
10. Why named policy conditions can be easier to review than one large expression.

---

## 22. Completion criteria

You have completed this topic when you can:

- write clear `if`/`elif`/`else` branches;
- reason about truth-value testing;
- combine conditions with Boolean operators;
- perform membership checks correctly;
- use `is None` appropriately;
- validate configuration before taking action;
- distinguish missing, false, zero, and empty values when required;
- use guard clauses to reject invalid states early;
- keep policy separate from operational actions;
- write conditions that another engineer can review without reconstructing hidden logic.

---

## Verified facts vs repository design

### Verified facts

The Python documentation defines conditional statements, truth-value testing, Boolean operations, comparisons, membership operations, and conditional expressions.

### Repository design decisions

This topic emphasizes guard clauses, explicit validation, named policy conditions, and separating policy from mechanism because the repository targets DevOps automation. These are teaching recommendations, not Python language requirements.

### Demonstrations

The deployment, health, replica, environment, and maintenance examples are intentionally small demonstrations. They do not claim behavior for a specific cloud, Kubernetes distribution, CI/CD system, or deployment platform.

---

## Authoritative references

- Python Tutorial — More Control Flow Tools: https://docs.python.org/3/tutorial/controlflow.html
- Python Built-in Types — Truth Value Testing: https://docs.python.org/3/library/stdtypes.html#truth-value-testing
- Python Built-in Types — Boolean Operations: https://docs.python.org/3/library/stdtypes.html#boolean-operations-and-or-not
- Python Expressions — Conditional expressions and Boolean operations: https://docs.python.org/3/reference/expressions.html
- Python Tutorial — An Informal Introduction to Python: https://docs.python.org/3/tutorial/introduction.html
