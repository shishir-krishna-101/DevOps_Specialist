# Modules, Packages, and Imports

## Why this topic matters

As Python automation grows beyond a single script, code needs boundaries that are easy to reuse, test, and maintain. **Modules** let you organize Python definitions in separate files. **Packages** provide a namespace for related modules and subpackages. **Imports** connect those pieces.

For DevOps automation, this matters when a project grows from one operational script into reusable components such as configuration handling, command execution, cloud helpers, API clients, and validation logic.

> **Verified fact:** A Python module can contain definitions and executable statements, and its module name is available through `__name__`.
>
> **Verified fact:** Python packages provide a way to structure module namespaces using dotted names such as `package.module`.
>
> **Repository design decision:** This guide uses small automation-oriented examples rather than reproducing large framework layouts.

---

## 1. What is a module?

A module is a Python file containing definitions and statements.

For example:

```text
project/
├── main.py
└── deployment.py
```

`deployment.py` can contain reusable functions:

```python
# deployment.py


def validate_environment(environment: str) -> bool:
    return environment in {"dev", "staging", "production"}
```

Another module can import it:

```python
# main.py

import deployment


if deployment.validate_environment("production"):
    print("environment accepted")
```

The important idea is that `main.py` does not need to duplicate the implementation.

**Source:** https://docs.python.org/3/tutorial/modules.html

---

## 2. Module namespaces

Every module has its own namespace. Names defined inside one module do not automatically become names in another module.

With:

```python
import deployment
```

the importing module can access names through the module name:

```python
deployment.validate_environment("staging")
```

This is often clearer than importing many individual names because the call site shows where the function comes from.

A module's name is available through `__name__`:

```python
print(deployment.__name__)
```

The value depends on how the module is being used.

---

## 3. `import` versus `from ... import ...`

Python supports several import forms.

### Import the module

```python
import deployment

deployment.validate_environment("production")
```

### Import a specific name

```python
from deployment import validate_environment

validate_environment("production")
```

### Import with an alias

```python
import deployment as deploy

deploy.validate_environment("production")
```

Or:

```python
from deployment import validate_environment as is_valid_environment
```

The choice should improve readability rather than merely reduce typing.

> **Common mistake:** Avoid aliases that make the source of an important operation unclear.

**Source:** https://docs.python.org/3/tutorial/modules.html

---

## 4. Avoid `from module import *`

Python supports wildcard imports:

```python
from deployment import *
```

The official tutorial notes that this can introduce an unknown set of names and make code harder to read. It is generally discouraged in normal program code.

Prefer explicit imports:

```python
from deployment import validate_environment
```

or import the module:

```python
import deployment
```

This makes dependencies easier to see during code review and maintenance.

**Source:** https://docs.python.org/3/tutorial/modules.html

---

## 5. Modules execute during import

A module can contain executable statements as well as function and class definitions.

For example:

```python
# settings.py

print("loading settings")

DEFAULT_REGION = "example-region"
```

Importing it executes its module-level code as part of loading the module.

This leads to an important automation rule:

> Keep module-level side effects small and intentional.

Avoid making an imported module unexpectedly:

- delete resources
- restart services
- make destructive API calls
- modify infrastructure
- launch long-running processes

A reusable module should normally expose functions, classes, or data that another part of the application can explicitly invoke.

---

## 6. Modules are normally imported once per interpreter session

Python maintains imported modules in `sys.modules`.

When a module has already been imported, subsequent imports can use the cached module object rather than loading and executing the module from scratch again.

You can inspect the cache:

```python
import sys

print("deployment" in sys.modules)
```

The import system documentation describes `sys.modules` as the cache checked during import processing.

This is useful for understanding why changing a module while an interactive Python session is running does not automatically cause the updated source to execute on every later `import`.

For interactive experimentation, Python provides `importlib.reload()` for reinitializing an already imported module.

```python
import importlib
import deployment

importlib.reload(deployment)
```

Use reloading deliberately; application code should not normally depend on repeatedly reloading modules.

**Source:** https://docs.python.org/3/reference/import.html

---

## 7. `__name__` and `__main__`

A Python file can be both an importable module and an executable script.

The standard pattern is:

```python
def main() -> None:
    print("running deployment workflow")


if __name__ == "__main__":
    main()
```

When the file is imported, its `__name__` is the module's import name, so the `main()` call does not run automatically.

When the file is executed as the main program, Python sets `__name__` to `"__main__"`, so the guarded code runs.

This gives a useful separation:

```text
Reusable definitions
        ↓
main()
        ↓
__name__ == "__main__"
        ↓
Execute CLI/script behavior
```

This pattern is particularly useful for DevOps scripts that should also expose reusable functions for tests or other automation.

**Source:** https://docs.python.org/3/tutorial/modules.html

---

## 8. Module search and `sys.path`

When Python imports a module, the import system has to locate it.

For common file-based imports, the search path is represented by `sys.path`.

You can inspect it with:

```python
import sys

for entry in sys.path:
    print(entry)
```

The Python tutorial documents several inputs used to initialize `sys.path`, including the directory containing the input script, `PYTHONPATH`, and installation-dependent defaults.

The exact environment matters, so do not assume that the current working directory is always the only relevant import location.

**Source:** https://docs.python.org/3/tutorial/modules.html

---

## 9. Why `sys.path` matters in DevOps projects

Import problems often appear when a script works from one directory but not another.

For example, a repository might contain:

```text
project/
├── scripts/
│   └── deploy.py
└── automation/
    └── deployment.py
```

If `deploy.py` depends on how the project happens to be launched, imports may behave differently than expected.

Do not solve import problems by blindly adding arbitrary directories to `sys.path`.

Instead, first understand:

1. What module name is being imported?
2. Where is Python searching?
3. How is the program being executed?
4. Is the project structure using a package?
5. Is the project intended to be installed as a package?

The import system is more precise than simply treating every import as a relative filesystem lookup.

---

## 10. Packages

A package organizes related modules under a dotted namespace.

For example:

```text
automation/
├── __init__.py
├── config.py
├── commands.py
└── validation.py
```

The modules can be referred to through names such as:

```text
automation.config
automation.commands
automation.validation
```

Python's official documentation describes packages as a way of structuring the module namespace.

Packages can also contain subpackages:

```text
automation/
├── __init__.py
├── cloud/
│   ├── __init__.py
│   └── aws.py
└── kubernetes/
    ├── __init__.py
    └── resources.py
```

This gives the project a hierarchy that can reflect its conceptual boundaries.

**Source:** https://docs.python.org/3/tutorial/modules.html#packages

---

## 11. Regular packages and `__init__.py`

Python's import documentation distinguishes regular packages from namespace packages.

A traditional regular package is typically a directory containing an `__init__.py` file:

```text
automation/
├── __init__.py
└── deployment.py
```

When a regular package is imported, its `__init__.py` is implicitly executed.

`__init__.py` can contain normal Python code, but keep package initialization intentional.

For example, a small package might expose a version or selected package-level object, but there is usually little reason to perform expensive or destructive work merely because a package was imported.

> **Verified fact:** Namespace packages are also supported by Python and do not require the same traditional `__init__.py` layout.
>
> **Repository design decision:** Examples in this repository use explicit package layouts where they make the learning model easier to understand.

**Source:** https://docs.python.org/3/reference/import.html

---

## 12. Importing a package module

Suppose the project contains:

```text
automation/
├── __init__.py
└── commands.py
```

A caller can import the module:

```python
import automation.commands
```

and then access it through the package namespace:

```python
automation.commands.run_command()
```

Another valid style is:

```python
from automation import commands

commands.run_command()
```

Or a specific name can be imported:

```python
from automation.commands import run_command

run_command()
```

Use the form that makes the dependency and ownership of the name clear.

---

## 13. Relative imports

Inside a package, imports can refer to modules relative to the current package.

For example:

```text
automation/
├── __init__.py
├── commands.py
└── deployment.py
```

`deployment.py` can use a relative import:

```python
from .commands import run_command
```

The leading dot means the import is relative to the current package.

A deeper package can use additional dots to refer to parent package levels where the package structure permits it.

Relative imports are useful when moving or distributing a package because internal dependencies can be expressed in terms of the package hierarchy rather than relying on a particular top-level installation path.

> **Common mistake:** A file executed directly as a standalone script may not have the same package context as when it is imported as part of a package. Understand how the program is launched before choosing relative imports.

**Source:** https://docs.python.org/3/tutorial/modules.html#intra-package-references

---

## 14. Absolute imports versus relative imports

Consider:

```text
automation/
├── __init__.py
├── commands.py
└── deployment.py
```

An absolute import can name the package explicitly:

```python
from automation.commands import run_command
```

A relative import can express the same internal relationship:

```python
from .commands import run_command
```

A useful repository guideline is:

```text
External dependency → explicit top-level/package import
Internal package relationship → choose a consistent package import style
```

This is a repository design guideline, not a Python requirement.

---

## 15. Packages are namespaces, not just folders

It is tempting to think:

```text
folder = package
file = module
```

That analogy is useful for learning, but the official import documentation explicitly warns against taking the filesystem analogy too literally.

Python modules and packages can be provided through import mechanisms that are not simply ordinary filesystem files and directories.

For everyday DevOps repositories, filesystem-backed packages are common and practical. For advanced import-system work, understand that Python's import machinery is more general.

**Source:** https://docs.python.org/3/reference/import.html

---

## 16. `importlib` for dynamic imports

Python's `importlib` module provides APIs for interacting with the import system.

For example:

```python
from importlib import import_module

module = import_module("automation.commands")
```

Dynamic imports can be useful when a module name is determined at runtime, but they should not replace ordinary imports without a reason.

For most application code, this is clearer:

```python
from automation import commands
```

Use dynamic importing when the application genuinely needs runtime module selection or plugin-like behavior.

**Source:** https://docs.python.org/3/reference/import.html

---

## 17. Importing and DevOps plugins

A plugin architecture might select an implementation based on configuration:

```text
Configuration
     ↓
Plugin name
     ↓
Dynamic import
     ↓
Implementation module
     ↓
Common interface
```

For example, a tool might choose between different notification backends.

The important design point is to keep the dynamic boundary narrow. Do not turn an entire codebase into string-based imports when a normal import would be clearer.

> **Repository design decision:** Plugin systems are presented as an advanced use case. They are not required for ordinary DevOps scripts.

---

## 18. Avoid circular imports

A circular dependency occurs when modules depend on each other in a cycle.

For example:

```text
module_a → module_b
    ↑         ↓
    └─────────┘
```

Circular imports can make initialization order difficult to understand and can produce partially initialized module state.

A better design is often to identify the shared concept and move it into a third module:

```text
module_a ──┐
           ↓
        common
           ↑
module_b ──┘
```

This is a design technique rather than a Python requirement, but it often produces cleaner automation projects.

---

## 19. Keep import-time work lightweight

Consider a module like:

```python
# cloud.py

connect_to_production()


def list_instances():
    ...
```

Importing the module now performs an external action immediately.

A safer design is:

```python
def connect_to_production():
    ...


def list_instances():
    ...
```

Then the caller explicitly controls when the external action occurs.

This is especially important for DevOps code because imports can happen indirectly through tests, command-line tools, web services, or other automation.

---

## 20. Module naming and collisions

Python searches for modules according to its import machinery and search path. A local file with a name matching an existing module can therefore create confusing behavior.

For example, naming a project file after a module that the application also intends to import can cause the local file to be selected unexpectedly depending on the search path.

Practical rule:

- choose descriptive project-specific module names
- avoid unnecessary names that collide with dependencies or standard-library modules
- inspect `module.__file__` when debugging where an import came from, where that attribute is available

Example:

```python
import json

print(json.__file__)
```

This can help diagnose an unexpected import source.

---

## 21. A maintainable DevOps package layout

A small automation project could evolve into a structure such as:

```text
ops_automation/
├── pyproject.toml
├── src/
│   └── ops_automation/
│       ├── __init__.py
│       ├── config.py
│       ├── commands.py
│       ├── logging_utils.py
│       └── deployment.py
└── tests/
    ├── test_config.py
    └── test_deployment.py
```

This is a **repository design example**, not a Python requirement.

The important lesson is the separation of responsibilities:

```text
config.py         → configuration model/loading
commands.py       → command execution boundary
logging_utils.py  → logging helpers
deployment.py     → deployment workflow
tests/            → verification
```

The exact project layout should be chosen according to the project's packaging and execution requirements.

---

## 22. Common mistakes

### Mistake 1: Putting everything in one script

A single script is fine for a small task, but large scripts become difficult to test and reuse.

### Mistake 2: Excessive wildcard imports

They hide where names come from and can make code harder to review.

### Mistake 3: Heavy import-time side effects

Importing a module should not unexpectedly modify infrastructure or perform expensive operations.

### Mistake 4: Blindly editing `sys.path`

Changing the search path can hide structural problems instead of fixing them.

### Mistake 5: Confusing execution mode with package imports

A module executed directly may have different package context from the same file imported as part of a package.

### Mistake 6: Circular dependencies

If two modules need each other, reconsider the boundaries and extract shared concepts where appropriate.

### Mistake 7: Assuming every directory is automatically the same kind of package

Python supports regular packages and namespace packages. Understand which model the project is using.

### Mistake 8: Overusing dynamic imports

Dynamic imports are powerful but add indirection. Prefer ordinary imports when the dependency is known at development time.

---

## 23. DevOps exercises

### Exercise 1 — Split a script

Take a small automation script and split it into:

- configuration
- validation
- command execution
- main workflow

Use modules and explicit imports.

### Exercise 2 — Script and module

Create a module containing a reusable function and a `main()` function. Use the `if __name__ == "__main__":` guard so the CLI behavior runs only when the file is executed directly.

### Exercise 3 — Build a package

Create:

```text
automation/
├── __init__.py
├── config.py
└── commands.py
```

Import `commands` from a separate program and call one of its functions.

### Exercise 4 — Relative import

Add a `deployment.py` module that imports a helper from `commands.py` using a relative import.

### Exercise 5 — Diagnose an import

Print `sys.path` and inspect the `__file__` attribute of an imported module. Explain why Python selected that module.

### Exercise 6 — Find a circular dependency

Create a deliberately circular pair of modules, observe the failure, then redesign the modules so their shared functionality lives in a third module.

---

## 24. Knowledge checks

You should be able to answer these without looking at the file:

1. What is a Python module?
2. What problem do packages solve?
3. What is the difference between `import module` and `from module import name`?
4. Why is `from module import *` generally discouraged?
5. What does `__name__ == "__main__"` mean?
6. What is `sys.path` used for?
7. What is `sys.modules` used for during importing?
8. What is the difference between a regular package and a namespace package?
9. Why can relative imports be useful inside a package?
10. Why are import-time side effects risky in DevOps automation?
11. What causes a circular import problem?
12. When might `importlib.import_module()` be appropriate?

---

## 25. Completion checklist

You are ready to move on when you can:

- [ ] Explain what a module is.
- [ ] Import a module and access its names.
- [ ] Use explicit imports and aliases appropriately.
- [ ] Explain why wildcard imports are usually avoided.
- [ ] Use `__name__ == "__main__"` correctly.
- [ ] Explain the role of `sys.path`.
- [ ] Explain the role of `sys.modules` at a high level.
- [ ] Create and import a package.
- [ ] Distinguish regular and namespace packages at a high level.
- [ ] Use relative imports inside a package.
- [ ] Recognize import-time side effects.
- [ ] Diagnose basic import-path problems.
- [ ] Recognize circular dependencies.
- [ ] Explain when dynamic imports are appropriate.

---

## 26. Verified facts vs repository design

### Verified facts

- Python modules contain definitions and statements and have their own namespaces.
- `__name__` identifies the module name in its execution context.
- Executing a module as the main program sets `__name__` to `"__main__"`.
- Python maintains imported modules in `sys.modules`.
- `sys.path` participates in locating modules.
- Packages structure module namespaces using dotted names.
- Python supports regular packages and namespace packages.
- `importlib` provides APIs for interacting with the import system.

### Repository design decisions

- Examples use DevOps automation concepts such as deployment, commands, and configuration.
- The recommended project boundaries are learning-oriented and are not mandatory Python architecture.
- The examples favor explicit, readable imports over wildcard imports.
- The repository treats destructive import-time side effects as a design smell for automation code.

---

## Authoritative references

- Python Tutorial — Modules: https://docs.python.org/3/tutorial/modules.html
- Python Tutorial — Packages: https://docs.python.org/3/tutorial/modules.html#packages
- Python Reference — The import system: https://docs.python.org/3/reference/import.html
- Python Standard Library — `importlib`: https://docs.python.org/3/library/importlib.html
