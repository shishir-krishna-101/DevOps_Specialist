# Functions

Functions are reusable units of behavior. In DevOps automation, they help turn a script into small, testable pieces of work with clear inputs, outputs, and failure behavior.

## 1. Mental Model

Think of a function as:

```text
input → work → result
```

Example:

```python
def normalize_environment(value):
    return value.strip().lower()
```

Calling the function:

```python
environment = normalize_environment(" Production ")
```

**Verified fact:** Python functions are defined with `def` and can accept parameters and return values.

**Repository design decision:** The examples focus on automation boundaries such as configuration, validation, command preparation, and result handling.

## 2. Defining and Calling Functions

A basic function:

```python
def check_health():
    print("health check")

check_health()
```

A function can accept parameters:

```python
def greet_service(service):
    print(f"checking {service}")

greet_service("api")
```

A function body is executed when the function is called, not merely because its definition appears in the file.

## 3. Parameters and Arguments

A parameter is a name in the function definition:

```python
def deploy(service):
    ...
```

An argument is the value supplied when calling it:

```python
deploy("api")
```

Multiple parameters can be defined:

```python
def build_endpoint(host, port):
    return f"https://{host}:{port}"
```

Call it with:

```python
endpoint = build_endpoint("api.example.internal", 443)
```

## 4. Return Values

Use `return` to send a result back to the caller:

```python
def is_production(environment):
    return environment == "production"

if is_production("production"):
    print("production policy")
```

A function can return multiple values by returning a tuple:

```python
def endpoint_parts():
    return "api.example.internal", 443

host, port = endpoint_parts()
```

A function that reaches the end without returning a value returns `None`.

Do not confuse displaying a value with returning it:

```python
def wrong():
    print("ok")


def right():
    return "ok"
```

The first function displays text; the second provides a value to its caller.

## 5. Default Arguments

Parameters can have default values:

```python
def retry_count(retries=3):
    return retries
```

The caller can use the default:

```python
retry_count()
```

Or provide another value:

```python
retry_count(5)
```

Defaults are evaluated when the function definition is executed. Avoid mutable objects such as lists or dictionaries as defaults unless their persistent shared state is deliberately intended.

Prefer:

```python
def add_host(host, hosts=None):
    if hosts is None:
        hosts = []
    hosts.append(host)
    return hosts
```

rather than using a mutable list as a default parameter.

## 6. Keyword Arguments

Arguments can be supplied by parameter name:

```python
def deploy(service, environment):
    ...

deploy(service="api", environment="staging")
```

Keyword arguments can make automation code easier to read when several values have similar meanings.

Python also supports positional arguments before keyword arguments, subject to the function's parameter rules.

## 7. Positional and Keyword-Only Design

A function can require some arguments to be supplied by keyword using `*`:

```python
def deploy(service, *, environment):
    ...
```

This requires:

```python
deploy("api", environment="staging")
```

The following is invalid because `environment` is keyword-only:

```python
# deploy("api", "staging")
```

Keyword-only parameters can make configuration-heavy automation interfaces clearer.

## 8. Positional-Only Design

Python also supports positional-only parameters using `/`:

```python
def connect(host, port, /):
    ...
```

These parameters must be supplied positionally.

This feature is useful when an API intentionally wants to separate positional interface design from keyword naming.

For most beginner DevOps functions, use ordinary parameters unless there is a clear reason to constrain how callers provide arguments.

## 9. `*args` and `**kwargs`

`*args` collects additional positional arguments:

```python
def show_services(*services):
    for service in services:
        print(service)

show_services("web", "api", "worker")
```

`**kwargs` collects additional keyword arguments:

```python
def show_config(**config):
    for key, value in config.items():
        print(key, value)

show_config(region="ap-south-1", retries=3)
```

These features are powerful, but do not use them merely to avoid defining a clear function interface.

## 10. Argument Forwarding

A wrapper can forward positional and keyword arguments:

```python
def wrapper(*args, **kwargs):
    return target(*args, **kwargs)
```

This pattern is common in decorators and adapter functions.

Use it deliberately. A function with an explicit interface is often easier to understand and validate than one that accepts arbitrary arguments.

## 11. Scope and Local Variables

Variables created inside a function are normally local to that function:

```python
def build_message():
    message = "ready"
    return message
```

The local name `message` is not automatically available outside the function.

Prefer passing values into functions and returning results rather than relying on mutable global state.

Less maintainable:

```python
config = {}


def configure():
    config["region"] = "ap-south-1"
```

More explicit:

```python
def configure(config):
    config["region"] = "ap-south-1"
    return config
```

The second design makes the data dependency visible to callers.

## 12. Functions as First-Class Objects

Functions can be assigned to names and passed to other functions:

```python
def normalize(value):
    return value.strip().lower()

processor = normalize
print(processor(" PROD "))
```

A function can accept another function:

```python
def apply_processor(value, processor):
    return processor(value)
```

This is useful when automation behavior varies while the surrounding workflow remains the same.

## 13. Lambda Functions

A lambda creates a small anonymous function:

```python
sort_key = lambda service: service["name"]
```

For simple transformations, lambdas can be useful as arguments to APIs such as sorting functions.

If the logic needs multiple statements or significant explanation, use a named function instead.

## 14. Functions for Validation

Validation logic is often a good candidate for a dedicated function:

```python
def validate_environment(environment):
    allowed = {"dev", "staging", "production"}
    if environment not in allowed:
        raise ValueError("unsupported environment")
    return environment
```

Then the main workflow can remain focused:

```python
environment = validate_environment(config["environment"])
```

A useful design principle is to validate at boundaries rather than repeatedly checking the same assumption throughout the workflow.

## 15. Functions for Transformation

Separate parsing or transformation from the action that consumes the result:

```python
def normalize_service_name(value):
    return value.strip().lower()

service = normalize_service_name(raw_service)
```

This makes the transformation independently testable.

A larger automation workflow can then look like:

```text
read input
   ↓
parse
   ↓
validate
   ↓
transform
   ↓
perform action
   ↓
verify result
   ↓
report
```

## 16. Functions for Side Effects

Some functions intentionally perform side effects such as writing a file, starting a process, or making a network request.

Keep side effects visible in the function's purpose:

```python
def write_report(path, content):
    with open(path, "w", encoding="utf-8") as file:
        file.write(content)
```

A function that performs an external action should not pretend that it is merely computing a value.

For automation code, it is useful to distinguish:

```text
pure transformation → input becomes output
side-effecting action → external state may change
```

## 17. Functions and Error Handling

Functions can raise exceptions when their preconditions are not satisfied:

```python
def require_region(config):
    if "region" not in config:
        raise ValueError("region is required")
    return config["region"]
```

Callers can decide where an exception should be handled.

Avoid catching exceptions inside every small function merely to return `None`. That can hide the actual failure and make automation harder to diagnose.

A useful boundary is often:

```text
low-level function
      ↓
raise useful exception
      ↓
workflow boundary
      ↓
log/report/choose exit behavior
```

## 18. Functions and Type Hints

Type annotations can document intended interfaces:

```python
def normalize_environment(value: str) -> str:
    return value.strip().lower()
```

They can make function contracts easier to read and can support tooling, but annotations do not by themselves validate runtime values.

More advanced typing is covered later in **Complete Python**.

## 19. Designing Small Automation Functions

Prefer functions with one clear responsibility.

Instead of one large function:

```text
load config
parse arguments
validate config
call cloud API
write files
send notification
```

separate the responsibilities when that improves clarity:

```text
load_config()
validate_config()
provision_resource()
write_report()
notify()
```

The exact boundaries are a repository/application design decision. Avoid splitting code into tiny functions that add more indirection than value.

## 20. Pure Functions vs Side Effects

A pure transformation can often be tested without infrastructure:

```python
def normalize_region(region):
    return region.strip().lower()
```

A side-effecting operation interacts with an external system:

```python
def write_report(path, content):
    ...
```

Keeping transformations separate from external actions can make automation easier to test.

For example:

```text
raw configuration
      ↓
parse/normalize function
      ↓
validated configuration
      ↓
side-effecting deployment function
```

## 21. Common Mistakes

### Mistake 1: Forgetting `return`

```python
def normalize(value):
    value.strip().lower()
```

The expression is evaluated but its result is not returned.

### Mistake 2: Confusing `print()` with `return`

Printing is output to the console; returning supplies a result to the caller.

### Mistake 3: Using mutable default arguments

```python
def add_item(item, items=[]):
    ...
```

The default object is reused across calls. Use `None` and create a new object when needed.

### Mistake 4: Excessive global state

Hidden dependencies make functions harder to test and reuse.

### Mistake 5: Catching every exception

Broad exception handling can hide failures and make operational debugging difficult.

### Mistake 6: Giant functions

A function that performs parsing, validation, infrastructure changes, reporting, and notification may be difficult to test and reason about.

### Mistake 7: Too many tiny functions

Abstraction has a cost. Split responsibilities when the resulting boundaries improve clarity, testing, or reuse.

### Mistake 8: Overusing `*args` and `**kwargs`

Flexible interfaces can make it harder to understand what a function actually requires.

## 22. Practical DevOps Example

A small configuration workflow:

```python
def normalize_environment(value):
    return value.strip().lower()


def validate_environment(environment):
    allowed = {"dev", "staging", "production"}
    if environment not in allowed:
        raise ValueError(f"unsupported environment: {environment}")
    return environment


def deployment_policy(environment):
    if environment == "production":
        return "approval-required"
    return "automatic"


raw_environment = " Production "
environment = normalize_environment(raw_environment)
environment = validate_environment(environment)
policy = deployment_policy(environment)

print(policy)
```

The demonstration intentionally separates normalization, validation, and policy selection. It does not represent a universal deployment policy for any particular platform.

## 23. Practical Exercises

### Exercise 1 — Environment validator

Write a function that accepts an environment name and raises an exception when it is not in an allowed set.

### Exercise 2 — Service normalizer

Write a function that strips whitespace, normalizes case, and returns a service name.

### Exercise 3 — Retry policy

Write a function that accepts an attempt number and returns whether another attempt is allowed according to a maximum retry count.

### Exercise 4 — Inventory lookup

Write a function that accepts a service name and dictionary and returns the corresponding host. Decide explicitly how missing services should be handled.

### Exercise 5 — Function composition

Create separate functions for parsing, validation, and normalization. Connect them into a small workflow.

### Exercise 6 — Side-effect boundary

Create one function that builds report content and another that writes it to a file. Explain why separating these responsibilities helps testing.

### Exercise 7 — Flexible arguments

Write a function using `*args` or `**kwargs`, then rewrite it with an explicit interface. Compare which version is easier to understand.

## 24. Knowledge Check

Before moving on, you should be able to answer:

1. What is the difference between a parameter and an argument?
2. What does `return` do?
3. What does a function return when it reaches the end without a return value?
4. Why can mutable default arguments cause problems?
5. What is the purpose of keyword-only parameters?
6. What are `*args` and `**kwargs`?
7. Why should automation functions avoid unnecessary global state?
8. What is the difference between a pure transformation and a side effect?
9. Where should exceptions usually be handled in a multi-step automation workflow?
10. Why can smaller, focused functions be easier to test?

## 25. Completion Criteria

You are ready to move on when you can:

- define and call functions confidently;
- use positional, keyword, default, and keyword-only arguments;
- explain positional-only parameters;
- return useful values from functions;
- avoid mutable default argument mistakes;
- use `*args` and `**kwargs` appropriately;
- pass functions as values when useful;
- separate validation, transformation, and side effects;
- design small functions around clear responsibilities;
- handle failures at appropriate workflow boundaries;
- build testable DevOps automation components.

## 26. Source Notes

### Verified facts

The Python language behavior described here is based on the official Python documentation.

### Repository design decisions

The DevOps workflow patterns, function boundaries, examples, exercises, and validation guidance are teaching choices for this repository. They are not requirements imposed by a particular cloud provider, CI system, Kubernetes, or automation platform.

### Authoritative references

- Python Tutorial — Defining Functions: https://docs.python.org/3/tutorial/controlflow.html#defining-functions
- Python Tutorial — More on Defining Functions: https://docs.python.org/3/tutorial/controlflow.html#more-on-defining-functions
- Python Language Reference — Function definitions: https://docs.python.org/3/reference/compound_stmts.html#function-definitions
- Python Language Reference — Expressions: https://docs.python.org/3/reference/expressions.html
- Python Standard Library — `functools`: https://docs.python.org/3/library/functools.html
