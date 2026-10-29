# Advanced Functions

Advanced functions are the next step after being comfortable with ordinary function definitions, arguments, return values, and basic scope.

This topic focuses on how Python functions behave as objects and how to design function interfaces that remain clear as programs become larger.

> **Prerequisite:** Complete the functions section of [Python for DevOps](../Python%20for%20DevOps/README.md) first. This file intentionally goes deeper instead of repeating the basics.

## 1. What you will learn

By the end of this topic, you should be able to:

- design functions with positional-only and keyword-only parameters
- understand functions as first-class objects
- pass functions as arguments and return functions from other functions
- explain lexical scope and closures
- use `nonlocal` when a closure needs to update an enclosing variable
- distinguish closures from objects when choosing a design
- understand how `*args` and `**kwargs` interact with advanced function interfaces
- write small higher-order functions without making code unnecessarily abstract

---

## 2. Positional and keyword arguments

Python lets callers provide arguments by position or by keyword.

```python
def connect(host, port=443, timeout=10):
    return host, port, timeout

connect("api.example.com")
connect("api.example.com", 8443)
connect("api.example.com", timeout=5)
```

For larger APIs, it can be useful to make the intended calling style explicit.

Python supports three parameter categories:

```text
positional-only | positional-or-keyword | keyword-only
```

The `/` and `*` markers in a function definition control these categories.

---

## 3. Positional-only parameters

A parameter before `/` is positional-only.

```python
def connect(host, /, port=443):
    return host, port
```

This is valid:

```python
connect("api.example.com", 8443)
```

This is not valid because `host` is positional-only:

```python
connect(host="api.example.com")
```

### Why use positional-only parameters?

Positional-only parameters can be useful when the parameter name is an implementation detail or when an API intentionally wants callers to provide the value positionally.

Do not add `/` simply because it exists. Use it when the interface benefits from that constraint.

---

## 4. Keyword-only parameters

A parameter after `*` is keyword-only.

```python
def deploy(service, *, environment, timeout=300):
    return service, environment, timeout
```

This is clear at the call site:

```python
deploy("payments", environment="production", timeout=600)
```

The following is invalid because `environment` must be passed by keyword:

```python
deploy("payments", "production")
```

### Why keyword-only parameters are useful

They make calls easier to read when a function has several optional settings.

For example:

```python
run_backup(
    "database-01",
    destination="s3",
    compression=True,
    retention_days=30,
)
```

The names communicate what each option means.

---

## 5. Combining the parameter styles

You can use all three categories in one function.

```python
def request(host, /, port=443, *, timeout=10, verify=True):
    return host, port, timeout, verify
```

Here:

- `host` is positional-only
- `port` is positional-or-keyword
- `timeout` and `verify` are keyword-only

This is especially useful when designing reusable libraries or internal APIs where you want the function's calling contract to be explicit.

---

## 6. First-class functions

Python functions are objects. They can be assigned to names, stored in collections, passed to other functions, and returned from functions.

```python
def square(value):
    return value * value

operation = square

print(operation(5))
```

`operation` now refers to the same function object as `square`.

### Functions in a collection

```python
def add(a, b):
    return a + b


def multiply(a, b):
    return a * b

operations = {
    "add": add,
    "multiply": multiply,
}

print(operations["add"](2, 3))
print(operations["multiply"](2, 3))
```

This pattern can be useful when a program needs to select behavior from a known set of operations.

For example, a CLI tool could map an operation name to a function rather than building a large chain of unrelated conditionals.

---

## 7. Passing functions to other functions

A function can accept another function as an argument.

```python
def apply_operation(value, operation):
    return operation(value)


def double(value):
    return value * 2

print(apply_operation(10, double))
```

Functions that accept or return other functions are often called **higher-order functions**.

The important idea is simple:

```text
function → data value
        → can be passed around
        → can be selected
        → can be returned
```

You do not need a functional-programming framework to use this feature. It is part of normal Python.

---

## 8. Returning functions

A function can create and return another function.

```python
def make_multiplier(factor):
    def multiply(value):
        return value * factor

    return multiply

triple = make_multiplier(3)
print(triple(10))
```

`make_multiplier()` returns the nested `multiply()` function.

The returned function continues to have access to `factor`.

That behavior leads to **closures**.

---

## 9. Scope and nested functions

A nested function can access names from its enclosing function.

```python
def make_checker(expected):
    def check(value):
        return value == expected

    return check

is_production = make_checker("production")

print(is_production("production"))
print(is_production("staging"))
```

The inner `check()` function can access `expected`, even after `make_checker()` has returned.

This is a closure.

---

## 10. Closures

A closure is a function together with the retained values from its enclosing lexical scope that it references.

A useful mental model is:

```text
make_checker("production")
        │
        └── creates check()
                │
                └── retains access to expected = "production"
```

Example:

```python
def make_prefixer(prefix):
    def add_prefix(value):
        return f"{prefix}{value}"

    return add_prefix

log_name = make_prefixer("log-")

print(log_name("app"))
print(log_name("worker"))
```

The returned function retains access to `prefix`.

### When closures are useful

Closures can be useful for small pieces of behavior that need to retain configuration without requiring a full class.

Examples include:

- creating configured callbacks
- adapting a function to a particular environment
- small reusable transformations
- encapsulating a small amount of state

Do not use a closure automatically. If the behavior has substantial state or many related operations, a class may communicate the design better.

---

## 11. `nonlocal`

A closure can read a value from an enclosing scope. If it needs to rebind that enclosing variable, `nonlocal` is used.

```python
def make_counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment

counter = make_counter()

print(counter())
print(counter())
print(counter())
```

Without `nonlocal`, assigning to `count` inside `increment()` would create a local binding instead of rebinding the enclosing function's `count`.

### Mental model

```text
make_counter()
    │
    ├── count = 0
    │
    └── increment()
          │
          └── nonlocal count → updates enclosing count
```

Use `nonlocal` deliberately. If state becomes complicated, consider whether an object would be clearer.

---

## 12. Closures versus classes

Both closures and classes can retain state.

### Closure

```python
def make_counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment
```

### Class

```python
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self):
        self.count += 1
        return self.count
```

Neither is universally better.

A closure can be concise when there is one small operation around a small amount of private state. A class can be easier to understand when the state has several related operations or needs a more explicit interface.

### Design exercise

Implement a retry counter both ways. Then ask:

- Which version is easier to test?
- Which version is easier to extend?
- Which version communicates the state more clearly?

---

## 13. `*args`

`*args` collects additional positional arguments into a tuple.

```python
def log_values(*values):
    for value in values:
        print(value)

log_values("server-01", "server-02", "server-03")
```

Inside the function, `values` is a tuple.

The name `args` is conventional but not required:

```python
def log_values(*items):
    for item in items:
        print(item)
```

### When to use it

Use variable positional arguments when the function genuinely accepts an arbitrary number of positional values.

Do not use `*args` merely to avoid defining a clear function signature.

---

## 14. `**kwargs`

`**kwargs` collects additional keyword arguments into a dictionary.

```python
def configure_service(**settings):
    for name, value in settings.items():
        print(name, value)

configure_service(
    timeout=30,
    retries=3,
    environment="production",
)
```

Inside the function, `settings` is a dictionary.

As with `*args`, the name `kwargs` is conventional rather than required.

---

## 15. Combining `*args` and `**kwargs`

A wrapper function often needs to forward arguments to another function.

```python
def run(operation, *args, **kwargs):
    return operation(*args, **kwargs)
```

The `*` and `**` in the call unpack the tuple and dictionary back into positional and keyword arguments.

For example:

```python
def connect(host, port=443, *, timeout=10):
    return host, port, timeout

result = run(
    connect,
    "api.example.com",
    timeout=5,
)
```

This pattern is important when writing wrappers and decorators.

---

## 16. Avoid overusing `**kwargs`

This is easy to write:

```python
def deploy(**options):
    ...
```

But it can hide the function's actual interface.

Compare it with:

```python
def deploy(service, *, environment, timeout=300, dry_run=False):
    ...
```

The second signature documents the expected inputs directly.

Prefer explicit parameters when the accepted options are known.

Use `**kwargs` when accepting an intentionally open-ended set of keyword arguments is part of the design.

---

## 17. Functions as configuration

A useful application of first-class functions is selecting behavior without duplicating control flow.

```python
def restart_service(service):
    return f"restart {service}"


def check_service(service):
    return f"check {service}"

handlers = {
    "restart": restart_service,
    "check": check_service,
}

operation = handlers["check"]
print(operation("api"))
```

This can make small command dispatchers easier to extend.

### Important boundary

Do not turn every conditional into a function dictionary. If a simple `if` or `match` clearly expresses the logic, keep it simple.

---

## 18. Lambda expressions

A `lambda` creates a small anonymous function.

```python
square = lambda value: value * value
print(square(5))
```

For many cases, a normal `def` is easier to read and document.

Lambdas are particularly common when a small function is passed directly as an argument.

```python
servers = [
    {"name": "api", "priority": 20},
    {"name": "worker", "priority": 10},
]

servers.sort(key=lambda server: server["priority"])
```

### Rule of thumb

If the function needs a name, multiple statements, documentation, or meaningful debugging behavior, use `def` instead of forcing it into a lambda.

---

## 19. `map`, `filter`, and explicit loops

Python provides built-in functions such as `map()` and `filter()` that work with iterables.

Example:

```python
values = [1, 2, 3, 4]

squares = map(lambda value: value * value, values)
```

These are useful tools, but readability matters more than using a functional construct simply because it exists.

For example, a comprehension can often communicate a transformation clearly:

```python
squares = [value * value for value in values]
```

When choosing between alternatives, ask which version is easiest for another engineer to understand and maintain.

---

## 20. Function design for automation code

When a Python script grows into a reusable tool, function design becomes increasingly important.

### Prefer functions that have a clear responsibility

Instead of one large function:

```text
load config
→ authenticate
→ call API
→ parse response
→ write file
→ print report
```

split the work into understandable boundaries:

```text
load_config()
      ↓
create_client()
      ↓
fetch_resources()
      ↓
transform_resources()
      ↓
write_report()
```

This is a **repository design recommendation**, not a Python language requirement.

### Keep external effects visible

Functions that call APIs, modify files, execute commands, or change infrastructure should make those effects understandable from their interface and name.

For example:

```python
def read_inventory(path):
    ...


def write_inventory(path, records):
    ...
```

is easier to reason about than a generic function such as:

```python
def process(data):
    ...
```

---

## 21. Common mistakes

### Mistake 1: Using mutable default arguments

Avoid this pattern:

```python
def add_host(host, hosts=[]):
    hosts.append(host)
    return hosts
```

The default list is created once when the function is defined, not as a fresh list for every call.

Prefer:

```python
def add_host(host, hosts=None):
    if hosts is None:
        hosts = []

    hosts.append(host)
    return hosts
```

### Mistake 2: Hiding everything behind `*args` and `**kwargs`

A flexible signature can become an undocumented API.

Prefer explicit parameters when the contract is known.

### Mistake 3: Making closures unnecessarily complex

Closures are useful, but deeply nested functions can be harder to understand than a small class or ordinary top-level functions.

### Mistake 4: Creating abstractions before they are needed

A function that is used once does not automatically need a generic callback system.

Start with the simplest design that clearly expresses the behavior.

### Mistake 5: Ignoring function metadata in wrappers

When writing decorators, use `functools.wraps()` so the wrapper preserves important metadata from the wrapped function.

This becomes especially important when debugging and inspecting decorated functions.

---

## 22. Practical exercise: command dispatcher

Build a small command dispatcher.

### Requirements

Create functions for:

- checking a service
- restarting a service
- retrieving service information

Then map command names to functions:

```text
check   → check_service
restart → restart_service
info    → service_info
```

The program should:

1. receive a command name
2. find the corresponding function
3. pass the service name to it
4. handle an unknown command cleanly

### Extension

Add a function that logs execution time around any command.

This exercise prepares you for the next topic: **decorators**.

---

## 23. Practical exercise: configurable health checker

Create a function factory:

```python
make_health_checker(expected_status)
```

It should return a function that accepts an observed status and returns whether it matches the expected status.

Example behavior:

```python
is_healthy = make_health_checker("healthy")

is_healthy("healthy")  # True
is_healthy("failed")   # False
```

Then extend the exercise so the returned function also counts how many checks have been performed.

Decide whether the closure remains clearer than a class once state becomes more involved.

---

## 24. Practical exercise: function-based retry policy

Create a function that accepts another function and a retry count.

Conceptually:

```text
retry(operation, attempts)
        │
        ├── call operation
        ├── if it succeeds → return result
        └── if it fails → retry according to the policy
```

For this learning exercise, use a local function rather than a real infrastructure operation.

Think carefully about:

- which exceptions should be retried
- when the final exception should be raised
- whether the retry count means total attempts or additional retries
- how the caller can understand the failure

Do not add retries blindly to production automation. Retrying the wrong operation can make an incident worse.

---

## 25. Knowledge check

You should be able to answer these without looking at the examples.

1. What makes a function a first-class object in Python?
2. What is the difference between positional-only and keyword-only parameters?
3. What does `/` mean in a function signature?
4. What does `*` mean in a function signature?
5. What does `*args` collect?
6. What does `**kwargs` collect?
7. What is a closure?
8. Why can a nested function access an enclosing variable?
9. When is `nonlocal` required?
10. When might a class be clearer than a closure?
11. Why should explicit function parameters often be preferred over unrestricted `**kwargs`?
12. Why is `functools.wraps()` useful in decorators?

If you cannot explain these concepts in your own words, revisit the relevant section and write a small example yourself.

---

## 26. What to build before moving on

Before moving to **Object-Oriented Python**, build at least one small program that uses:

- a function with keyword-only parameters
- a function passed as an argument
- a function returned from another function
- a closure
- `*args` or `**kwargs` where they are genuinely useful
- a small function-dispatch table

The goal is not to use every feature at once. The goal is to understand when each feature improves a design and when it adds unnecessary complexity.

---

## Authoritative references

The explanations and syntax in this topic are grounded primarily in the official Python documentation:

- Python Tutorial — Defining Functions: https://docs.python.org/3/tutorial/controlflow.html#defining-functions
- Python Tutorial — More Control Flow Tools: https://docs.python.org/3/tutorial/controlflow.html
- Python Language Reference — Function definitions: https://docs.python.org/3/reference/compound_stmts.html#function-definitions
- Python Built-in Functions: https://docs.python.org/3/library/functions.html

For the exact behavior of a Python version you are using, consult the corresponding official Python documentation rather than relying on this learning summary alone.
