# Modules and Imports

Modules and imports let Python automation code be organized into reusable units. For DevOps work, this helps separate configuration, validation, API interaction, reporting, and other responsibilities without putting an entire workflow into one script.

## 1. Mental Model

Think about a module as a Python file that provides a namespace:

```text
module
  ↓
namespace
  ↓
names such as functions, classes, and variables
```

An import makes names from another module available to the importing code according to Python's import rules.

**Verified fact:** A Python module is a unit of organization containing Python definitions and statements. The `import` statement introduces names from a module into the importing namespace according to the import mechanism.

**Repository design decision:** The examples use DevOps-style modules such as configuration, validation, and reporting to demonstrate practical organization.

## 2. A Simple Module

Suppose a file named `config_utils.py` contains:

```python
def normalize_environment(value):
    return value.strip().lower()
```

Another file can import the module:

```python
import config_utils

environment = config_utils.normalize_environment(" Production ")
```

The module name is used as a namespace, which makes the origin of the function explicit.

## 3. `import`

The basic form is:

```python
import pathlib
```

You can then access names through the module:

```python
path = pathlib.Path("config.json")
```

This style is often useful when several imported modules expose names with similar meanings.

## 4. `from ... import ...`

Python also supports importing selected names:

```python
from pathlib import Path

path = Path("config.json")
```

This can make code shorter, but the imported name no longer visibly includes its module namespace.

Use it when the resulting name remains clear in context.

## 5. Import Aliases

An imported module can be given an alias:

```python
import pathlib as pl
```

Then:

```python
path = pl.Path("config.json")
```

Aliases can be useful when a long module name would otherwise make code difficult to read. Avoid arbitrary aliases that make the code harder for another engineer to understand.

## 6. Wildcard Imports

Python supports wildcard imports:

```python
from module_name import *
```

This introduces names from the module into the current namespace.

For maintainable automation code, explicit imports are generally easier to review because a reader can see where names come from.

Avoid wildcard imports when they make dependencies or name ownership unclear.

## 7. Modules Are Namespaces

A module can contain multiple names:

```python
# deployment_policy.py

DEFAULT_TIMEOUT = 30


def validate_environment(environment):
    allowed = {"dev", "staging", "production"}
    if environment not in allowed:
        raise ValueError("unsupported environment")
```

Another module can use:

```python
import deployment_policy

if deployment_policy.DEFAULT_TIMEOUT > 0:
    deployment_policy.validate_environment("staging")
```

The module namespace keeps related names together.

## 8. Import-Time Execution

A module's top-level statements execute when the module is imported, subject to Python's import machinery and module caching behavior.

This means top-level side effects should be treated carefully.

Avoid making an imported utility module unexpectedly:

- modify infrastructure;
- delete files;
- start long-running processes;
- make network calls;
- change production state.

Prefer definitions at module level and explicit function calls for actions.

For example, prefer:

```python
def deploy():
    # perform deployment work
    ...
```

rather than performing the deployment merely because the module was imported.

## 9. `__name__` and Script Entry Points

A common pattern allows a module to be imported without automatically running its command-line workflow:

```python
def main():
    print("deployment workflow")


if __name__ == "__main__":
    main()
```

When the file is run as the main program, the guarded block can invoke `main()`.

When the file is imported, the guarded block is not executed as the main-program entry point.

This pattern is useful for keeping reusable functions importable while still allowing the same file to act as a script.

## 10. `sys.modules`

Python maintains imported modules in `sys.modules`.

This is part of Python's import machinery and helps explain why repeatedly importing the same module in a process does not normally execute the module from scratch each time.

For normal application code, you usually do not need to manipulate `sys.modules` directly.

The important engineering lesson is that imports have process-level state and should not be treated as if every `import` were an isolated file execution.

## 11. `sys.path`

Python searches locations for importable modules using its import path.

The path can be inspected through `sys.path`:

```python
import sys

for entry in sys.path:
    print(entry)
```

The exact entries depend on how Python was started and the environment in which it is running.

Do not assume that a file is importable merely because it exists somewhere on disk. Its location must be discoverable through Python's import system.

## 12. Packages

A package organizes related modules into a larger namespace.

A simple project might look like:

```text
automation/
├── main.py
└── deployment/
    ├── __init__.py
    ├── config.py
    └── validation.py
```

Code can import from the package:

```python
from deployment.validation import validate_environment
```

Packages are useful when a project grows beyond a small number of modules.

## 13. `__init__.py`

A package directory may contain an `__init__.py` file.

For regular packages, this file is executed when the package is imported and can define package-level names or initialization behavior.

Modern Python also supports namespace packages, which can exist without an `__init__.py` under the conditions described by the Python import system.

For beginner DevOps projects, the important distinction is:

```text
module  → usually one .py file
package → namespace containing related modules
```

Do not assume every directory containing Python files automatically behaves identically to every other package layout.

## 14. Absolute Imports

An absolute import identifies the module from the importable package namespace:

```python
from deployment.validation import validate_environment
```

This is often straightforward for application projects because the source of the imported name is explicit.

## 15. Relative Imports

A package can use relative imports to refer to modules within the same package:

```python
from .validation import validate_environment
```

The leading dot expresses a relative relationship to the current package.

Relative imports are useful inside package code, but their behavior depends on the package context in which the module is executed.

## 16. Running a Package Module

A common source of confusion is the difference between importing a package module and executing a file directly.

For example, code containing a relative import may work when executed as part of its package but fail when the file is run directly because the package context is different.

For project-level automation, choose and document a consistent execution model instead of switching randomly between direct file execution and package execution.

## 17. Importing Standard Library Modules

Python includes a large standard library.

Examples commonly useful in DevOps automation include:

```python
import json
import logging
import pathlib
import subprocess
import sys
```

These modules cover different responsibilities. Keep imports explicit and organize code around the actual functionality required by the program.

## 18. `importlib`

Python provides `importlib` for programmatic interaction with the import system.

For example:

```python
import importlib

module = importlib.import_module("json")
```

Dynamic importing can be useful for plugin systems or situations where the module name is determined at runtime.

It also adds complexity. Do not use dynamic imports when an ordinary explicit import communicates the dependency clearly.

## 19. Circular Imports

A circular import can occur when modules depend on each other during import:

```text
module_a → module_b
     ↑         ↓
     └─────────┘
```

For example, if `a.py` imports `b.py` while `b.py` imports `a.py`, import-time dependencies can become difficult to resolve and reason about.

A common improvement is to move shared functionality into a third module:

```text
module_a ──┐
           ├──> common.py
module_b ──┘
```

This is a design technique, not a universal requirement. The goal is to make dependency direction understandable.

## 20. Import-Time Side Effects

Import-time work is especially important in automation because an innocent-looking import can otherwise trigger external behavior.

Avoid patterns such as:

```python
# bad design for a reusable utility module
connect_to_production()
delete_old_resources()
```

Prefer:

```python
def cleanup_resources():
    delete_old_resources()
```

Then invoke the operation explicitly from the workflow that owns the side effect.

## 21. Module-Level Configuration

Constants can be defined at module scope:

```python
DEFAULT_RETRIES = 3
DEFAULT_TIMEOUT = 30
```

This can be appropriate for stable defaults.

Do not confuse module constants with runtime configuration. Environment-specific or secret configuration should normally come from an explicit configuration boundary rather than being hard-coded into reusable modules.

## 22. A Practical DevOps Package Layout

A small automation project might use:

```text
automation/
├── main.py
└── deployment/
    ├── __init__.py
    ├── config.py
    ├── validation.py
    ├── runner.py
    └── report.py
```

Possible responsibilities:

```text
config.py      → load and normalize configuration
validation.py  → validate inputs and policy
runner.py      → perform external actions
report.py      → build or write reports
main.py        → orchestrate the workflow
```

**Repository design decision:** This layout is a teaching example, not an official Python or DevOps project requirement.

## 23. Keep Imports at Clear Boundaries

A useful structure is:

```text
main workflow
    ↓
configuration
    ↓
validation
    ↓
action layer
    ↓
reporting
```

Each module should have a clear responsibility and dependency direction where practical.

Do not create modules merely to increase the number of files. Split code when the separation improves clarity, testing, reuse, or ownership.

## 24. Common Mistakes

### Mistake 1: Wildcard imports everywhere

They can make it difficult to determine where a name came from.

### Mistake 2: Heavy work during import

Importing a helper should not unexpectedly modify external infrastructure.

### Mistake 3: Confusing a file path with an importable module

Python's import system searches its configured import path; arbitrary filesystem locations are not automatically importable.

### Mistake 4: Circular dependencies

Mutual module dependencies can make imports fragile and difficult to understand.

### Mistake 5: Mixing execution models

A module designed for package execution may behave differently when run directly as a file, especially when relative imports are involved.

### Mistake 6: Excessive module splitting

Too many tiny modules can increase navigation and dependency complexity without improving the design.

### Mistake 7: Hiding dependencies

Dynamic imports and broad wildcard imports can make the real dependency graph harder to review.

## 25. Practical Exercises

### Exercise 1 — Create a utility module

Create `environment.py` with a function that normalizes environment names. Import it from another script.

### Exercise 2 — Separate validation

Move validation functions into `validation.py` and import them from a workflow module.

### Exercise 3 — Script entry point

Create a `main()` function and use the `if __name__ == "__main__":` pattern. Verify that importing the module does not invoke the workflow automatically.

### Exercise 4 — Package structure

Create a small package with separate configuration and validation modules. Import both from a main workflow.

### Exercise 5 — Circular dependency analysis

Create two modules with a deliberately circular dependency, observe the failure, and refactor shared behavior into a third module.

### Exercise 6 — Inspect the import path

Print `sys.path` and explain which entries make your project modules importable.

### Exercise 7 — Import-time safety review

Review an automation module and identify any network calls, file mutations, or infrastructure actions that occur merely because the module is imported. Move those actions behind explicit functions where appropriate.

## 26. Knowledge Check

Before moving on, you should be able to answer:

1. What is a Python module?
2. What is the difference between `import module` and `from module import name`?
3. Why can explicit imports improve maintainability?
4. What does `__name__ == "__main__"` help control?
5. What are `sys.modules` and `sys.path` used for?
6. What is the difference between a module and a package?
7. What role can `__init__.py` play?
8. What is the difference between absolute and relative imports?
9. Why can circular imports be a problem?
10. Why should reusable automation modules avoid unexpected import-time side effects?

## 27. Completion Criteria

You are ready to move on when you can:

- create and import Python modules;
- distinguish module namespaces from imported names;
- use explicit imports appropriately;
- explain the purpose of `__name__` and the main-entry pattern;
- inspect the import path when diagnosing import problems;
- organize related modules into packages;
- understand absolute and relative imports;
- identify circular-import risks;
- keep external side effects behind explicit workflow boundaries;
- organize a small DevOps automation project into understandable modules.

## 28. Source Notes

### Verified facts

The Python language and import behavior described here is based on the official Python documentation.

### Repository design decisions

The package layouts, DevOps responsibilities, exercises, and module-boundary recommendations are teaching choices for this repository. They are not requirements imposed by Python, Kubernetes, a cloud provider, or a CI platform.

### Authoritative references

- Python Tutorial — Modules: https://docs.python.org/3/tutorial/modules.html
- Python Tutorial — Packages: https://docs.python.org/3/tutorial/modules.html#packages
- Python Language Reference — The import system: https://docs.python.org/3/reference/import.html
- Python Language Reference — Simple statements (`import`): https://docs.python.org/3/reference/simple_stmts.html#import
- Python Standard Library — `importlib`: https://docs.python.org/3/library/importlib.html
- Python Standard Library — `sys`: https://docs.python.org/3/library/sys.html
