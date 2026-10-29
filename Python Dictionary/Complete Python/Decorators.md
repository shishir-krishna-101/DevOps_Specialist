# Decorators

Decorators are a Python feature for building reusable behavior around functions, methods, and other callable objects. They are especially useful when the same cross-cutting behavior—such as logging, timing, validation, retries, or access checks—needs to be applied consistently.

This topic builds on **Advanced Functions** and prepares you for later topics such as context managers, testing, and advanced Python design.

> **Source note:** The language behavior described here follows the official Python documentation. Examples marked as demonstrations are repository teaching examples, not claims about an external system.

## 1. The core mental model

A decorator is a callable that takes another callable and returns a callable or other replacement object.

Conceptually:

```text
original function
       │
       ▼
   decorator
       │
       ▼
wrapped/replacement callable
```

For example:

```python
def add_logging(function):
    def wrapper(*args, **kwargs):
        print(f"calling {function.__name__}")
        return function(*args, **kwargs)

    return wrapper
```

The decorator receives `function`, creates `wrapper`, and returns `wrapper`.

## 2. Functions are objects

Decorators depend on Python functions being first-class objects.

A function can be assigned to another name:

```python
def deploy():
    print("deploying")

run = deploy
run()
```

A function can also be passed as an argument:

```python
def execute(operation):
    return operation()
```

Because functions can be passed around, another function can accept a function, add behavior, and return a replacement callable.

## 3. Applying a decorator manually

Before using decorator syntax, understand the equivalent assignment.

```python
def greet(name):
    return f"Hello, {name}"


def add_logging(function):
    def wrapper(*args, **kwargs):
        print("calling function")
        return function(*args, **kwargs)

    return wrapper


greet = add_logging(greet)
```

After this assignment, the name `greet` refers to the returned wrapper.

Calling it runs the additional behavior and then the original function:

```python
print(greet("Alex"))
```

This transformation is the foundation of decorator syntax.

## 4. The `@decorator` syntax

Python provides decorator syntax for applying a decorator to a function definition.

```python
def add_logging(function):
    def wrapper(*args, **kwargs):
        print(f"calling {function.__name__}")
        return function(*args, **kwargs)

    return wrapper


@add_logging
def deploy():
    print("deploying")
```

The decorated function can then be called normally:

```python
deploy()
```

The decorator syntax is equivalent in effect to replacing the function name with the result of applying the decorator.

> **Verified fact:** Python's function-definition grammar permits decorators, and the decorator expressions are evaluated when the function definition is executed.

## 5. Decorators execute at function definition time

A common misunderstanding is that the decorator itself runs every time the decorated function is called.

The decorator expression is evaluated and the resulting decorator is applied when the function definition executes.

For example:

```python
def announce(function):
    print("applying decorator")
    return function


@announce
def task():
    print("running task")
```

The message from `announce()` occurs when the definition is processed, not because `task()` was called.

The returned callable is what is later invoked when `task()` is called.

## 6. The wrapper runs at call time

A typical decorator has two different moments:

```text
function definition time
        │
        └── decorator is applied

function call time
        │
        └── wrapper executes
             │
             └── original function may execute
```

Example:

```python
def log_calls(function):
    def wrapper(*args, **kwargs):
        print(f"starting {function.__name__}")
        result = function(*args, **kwargs)
        print(f"finished {function.__name__}")
        return result

    return wrapper
```

The wrapper's body runs when the decorated callable is called.

## 7. Preserve arguments with `*args` and `**kwargs`

A general-purpose wrapper often needs to accept the same arguments as the wrapped function.

```python
def log_calls(function):
    def wrapper(*args, **kwargs):
        print(f"calling {function.__name__}")
        return function(*args, **kwargs)

    return wrapper
```

This pattern forwards positional and keyword arguments to the original function.

It is useful when the decorator should work with multiple functions having different signatures.

> **Design note:** A generic `*args, **kwargs` wrapper is convenient, but it does not by itself document the wrapped function's signature. Use it deliberately rather than treating it as a substitute for clear APIs.

## 8. Return the wrapped function's result

A wrapper should normally return the result of the wrapped callable when the decorator is intended only to add behavior around the call.

```python
def log_calls(function):
    def wrapper(*args, **kwargs):
        print("starting")
        result = function(*args, **kwargs)
        print("finished")
        return result

    return wrapper
```

Forgetting the `return` can silently change the behavior of the decorated function:

```python
def broken(function):
    def wrapper(*args, **kwargs):
        function(*args, **kwargs)
        # Missing return

    return wrapper
```

If the original function returned a value, the broken wrapper does not preserve it.

## 9. Closures and decorators

Decorators commonly use closures.

The wrapper function can refer to `function` from the enclosing `add_logging()` call:

```python
def add_logging(function):
    def wrapper(*args, **kwargs):
        print(function.__name__)
        return function(*args, **kwargs)

    return wrapper
```

Here, `function` is not a local variable of `wrapper`. It comes from the enclosing scope.

This connects decorators directly to the previous **Scopes and Python Data Model** and **Advanced Functions** topics.

## 10. A practical logging decorator

A simple teaching example:

```python
def log_operation(function):
    def wrapper(*args, **kwargs):
        print(f"starting: {function.__name__}")
        result = function(*args, **kwargs)
        print(f"completed: {function.__name__}")
        return result

    return wrapper


@log_operation
def validate_configuration(path):
    return path
```

This demonstrates the structure of a cross-cutting concern without requiring a particular logging framework.

For production applications, prefer Python's `logging` module over `print()` for application logging.

## 11. `functools.wraps`

A wrapper is a different function object from the original function.

Without additional handling, useful metadata such as the wrapped function's name and docstring can be replaced by metadata belonging to the wrapper.

The standard library provides `functools.wraps()` for this purpose.

```python
from functools import wraps


def log_operation(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        print(f"calling {function.__name__}")
        return function(*args, **kwargs)

    return wrapper
```

`wraps()` is a convenience function for updating the wrapper function so that it resembles the wrapped function for relevant introspection metadata.

This is an important production-quality habit when writing decorators.

## 12. Why metadata matters

Consider a function with a useful docstring:

```python
def deploy():
    """Deploy the application."""
```

If it is replaced by a wrapper without preserving metadata, tools that inspect the resulting callable may see the wrapper rather than the original function.

Using `@wraps(function)` helps preserve the relevant metadata expected by introspection and documentation tooling.

> **Verified fact:** `functools.wraps()` is implemented in terms of `functools.update_wrapper()` and is intended to simplify writing well-behaved wrapper functions.

## 13. Decorators that accept configuration

A decorator can itself be created by another function when configuration is required.

This produces three layers:

```text
configuration
     │
     ▼
decorator factory
     │
     ▼
decorator
     │
     ▼
wrapper
```

Example:

```python
from functools import wraps


def announce(prefix):
    def decorator(function):
        @wraps(function)
        def wrapper(*args, **kwargs):
            print(f"{prefix}: {function.__name__}")
            return function(*args, **kwargs)

        return wrapper

    return decorator


@announce("DEPLOY")
def deploy():
    print("deploying")
```

Here:

- `announce("DEPLOY")` creates a decorator;
- that decorator receives `deploy`;
- the decorator creates the wrapper;
- the wrapper runs when `deploy()` is called.

## 14. Decorator arguments versus function arguments

This distinction is important:

```python
@announce("DEPLOY")
def deploy(environment):
    ...
```

The value `"DEPLOY"` configures the decorator.

The value passed to `deploy(environment)` is an argument to the decorated function.

They belong to different stages of the process.

## 15. Multiple decorators

More than one decorator can be applied to a function.

```python
@outer
@inner
def task():
    ...
```

Decorator application is nested: the result of applying the lower decorator becomes the input to the upper decorator.

Conceptually:

```python
task = outer(inner(task))
```

This means decorator order matters.

For example, if two decorators perform logging, validation, transformation, or error handling, changing their order can change the behavior.

## 16. Decorators on methods

Decorators are not limited to standalone functions.

They can also be applied to methods:

```python
from functools import wraps


def log_method(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        print(f"calling {function.__name__}")
        return function(*args, **kwargs)

    return wrapper


class Deployer:
    @log_method
    def deploy(self, environment):
        return f"deploying to {environment}"
```

The wrapper must allow the method's arguments to be forwarded correctly, including the instance passed as `self`.

## 17. Class decorators

Decorator syntax can also be used with classes.

A class decorator receives the class object and can return a class or another replacement object.

A simple demonstration:

```python
def mark_class(cls):
    cls.is_automation_component = True
    return cls


@mark_class
class Deployer:
    pass
```

This is a repository teaching example. Whether modifying a class this way is appropriate depends on the design of the application.

## 18. Decorators and exceptions

Decorators can centralize exception-handling behavior when that behavior is genuinely shared.

```python
from functools import wraps


def report_errors(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        try:
            return function(*args, **kwargs)
        except Exception as exc:
            print(f"{function.__name__} failed: {exc}")
            raise

    return wrapper
```

The important detail is `raise` after reporting the exception. The decorator records the failure without silently converting a failure into apparent success.

For production systems, use structured logging and make the exception-handling policy explicit.

## 19. Decorators and timing

Timing is another common cross-cutting concern.

```python
from functools import wraps
from time import perf_counter


def measure_time(function):
    @wraps(function)
    def wrapper(*args, **kwargs):
        start = perf_counter()
        try:
            return function(*args, **kwargs)
        finally:
            elapsed = perf_counter() - start
            print(f"{function.__name__}: {elapsed:.6f}s")

    return wrapper
```

Using `finally` ensures the timing calculation is reached whether the wrapped function returns normally or raises an exception.

This is a teaching example, not a benchmark methodology.

## 20. Decorators and DevOps automation

Decorators can be useful when multiple automation functions share a policy or cross-cutting concern.

Potential applications include:

- logging function entry and completion;
- measuring execution duration;
- validating inputs;
- normalizing error reporting;
- checking a precondition;
- recording metrics;
- attaching metadata to functions.

Use decorators when the shared behavior is clear and improves the design.

Do not use a decorator merely because a behavior can technically be implemented as one.

## 21. Decorators versus explicit code

A decorator can make repeated behavior concise:

```python
@require_configuration
@log_operation
def deploy():
    ...
```

But the behavior can become harder to discover if too many decorators are stacked together.

A good engineering rule is to favor clarity over cleverness.

Ask:

1. Is this behavior genuinely cross-cutting?
2. Does applying it declaratively improve readability?
3. Is decorator ordering obvious?
4. Can a new engineer understand what happens when the function is called?
5. Does the decorator preserve useful function behavior and metadata?

## 22. Common mistakes

### Mistake 1: Forgetting to return the wrapper

```python
def decorator(function):
    def wrapper(*args, **kwargs):
        return function(*args, **kwargs)

    # Missing: return wrapper
```

The decorator then returns `None` rather than the replacement callable.

### Mistake 2: Forgetting to return the wrapped result

```python
def decorator(function):
    def wrapper(*args, **kwargs):
        function(*args, **kwargs)

    return wrapper
```

The original result is discarded.

### Mistake 3: Forgetting `@wraps`

The decorated callable may expose wrapper metadata instead of the original function's metadata.

### Mistake 4: Swallowing exceptions

Avoid:

```python
try:
    function()
except Exception:
    pass
```

Silently hiding failures is especially dangerous in automation and deployment code.

### Mistake 5: Unexpected decorator order

With multiple decorators, changing their order can change behavior.

### Mistake 6: Doing too much inside a decorator

A decorator that silently performs unrelated infrastructure changes, modifies global state, or hides expensive work can make code difficult to reason about.

## 23. Testing decorated functions

Decorators should be tested as behavior, not just as syntax.

For a logging decorator, test questions such as:

- Does the wrapped function still execute?
- Are positional arguments forwarded?
- Are keyword arguments forwarded?
- Is the original return value preserved?
- Are exceptions propagated according to the intended policy?
- Is useful function metadata preserved?

For a timing decorator, test that the wrapped function's result and exception behavior remain correct.

The exact test strategy depends on the decorator's contract.

## 24. Practical exercises

### Exercise 1 — Basic logging decorator

Create a decorator that reports the name of a function before calling it.

Requirements:

- accept arbitrary positional and keyword arguments;
- call the original function;
- return its result;
- preserve metadata with `functools.wraps`.

### Exercise 2 — Execution timing

Create a decorator that measures execution duration.

Requirements:

- use a monotonic performance counter suitable for measuring elapsed time;
- preserve the original return value;
- report elapsed time even if the wrapped function raises.

### Exercise 3 — Required environment

Create a configurable decorator that checks whether a required environment variable is available before executing an operation.

Decide explicitly:

- what counts as missing;
- what exception should be raised;
- whether the original function is called at all when validation fails.

### Exercise 4 — Retry design review

Design a retry decorator on paper before implementing it.

Define:

- which exceptions are retryable;
- maximum attempts;
- delay/backoff policy;
- timeout behavior;
- what happens after the final failure.

Do not assume that retrying every exception is safe.

### Exercise 5 — Multiple decorators

Create two simple decorators and apply them in both possible orders. Observe and explain the difference.

## 25. Knowledge checks

1. What is a decorator?
2. Why do decorators depend on functions being first-class objects?
3. What does `@decorator` do conceptually?
4. When is the decorator applied relative to calling the decorated function?
5. Why does a wrapper commonly use `*args` and `**kwargs`?
6. Why should a transparent wrapper normally return the wrapped function's result?
7. What problem does `functools.wraps` address?
8. Why does decorator order matter?
9. How does a decorator factory differ from a decorator?
10. What risks arise when decorators hide too much behavior?

## 26. Completion checklist

You are ready to move on when you can:

- explain decorators without treating them as magic;
- manually transform a function using a decorator;
- use `@decorator` syntax correctly;
- write a wrapper with `*args` and `**kwargs`;
- preserve return values;
- use `functools.wraps`;
- explain decorator factories;
- explain multiple-decorator ordering;
- decorate methods and classes;
- design exception and timing decorators without hiding failures;
- recognize when a decorator improves a design and when explicit code is clearer.

## 27. Verified facts vs repository design

### Verified facts

The official Python documentation establishes the semantics of function decorators, including decorator expressions and their application to function definitions. The standard library documents `functools.wraps()` and `update_wrapper()` for preserving wrapper metadata.

### Repository design decisions

This repository uses logging, timing, validation, and DevOps automation examples because they make the decorator pattern relevant to the target audience.

These examples are teaching patterns, not official recommendations for a specific DevOps platform or production architecture.

## 28. Authoritative references

- Python glossary — Decorator: https://docs.python.org/3/glossary.html#term-decorator
- Python language reference — Function definitions: https://docs.python.org/3/reference/compound_stmts.html#function-definitions
- Python `functools` documentation: https://docs.python.org/3/library/functools.html
- `functools.wraps`: https://docs.python.org/3/library/functools.html#functools.wraps
- Python tutorial — More on defining functions: https://docs.python.org/3/tutorial/controlflow.html#more-on-defining-functions
