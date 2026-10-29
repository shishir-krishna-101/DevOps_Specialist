# Running Python

Running Python is more than knowing how to type `python` and execute a file. For DevOps work, you should understand the difference between interactive execution, running a script, passing arguments, and importing a module.

This topic establishes the execution model used by the rest of the Python-for-DevOps track.

---

## 1. The Python interpreter

Python programs are executed by the Python interpreter available in your environment.

The official Python tutorial documents two useful modes:

- **Interactive mode** — enter Python statements and see their results immediately.
- **Script execution** — give Python a source file to execute.

The exact command used to invoke Python can depend on the operating system and installation. Use the interpreter command provided by your environment rather than assuming a platform-specific command.

### Interactive example

When an interactive interpreter is started, you can experiment directly:

```text
>>> 2 + 3
5
>>> name = "devops"
>>> name.upper()
'DEVOPS'
```

Interactive mode is useful for:

- learning syntax;
- checking how an expression behaves;
- inspecting a value or object;
- testing a small idea before putting it into a script.

It is not a substitute for version-controlled automation code.

---

## 2. Python script files

A Python source file normally uses the `.py` extension.

Example:

```python
print("Hello, DevOps")
```

Save the program as `hello.py` and execute it with the Python interpreter available in your environment.

The important mental model is:

```text
Python interpreter
        ↓
Python source file
        ↓
Python statements execute
```

For DevOps automation, keep executable logic in files that can be reviewed, tested, version-controlled, and reproduced.

---

## 3. A clear script entry point

A reusable Python module can contain functions while also supporting direct execution.

A common pattern is:

```python
def main():
    print("Starting automation")


if __name__ == "__main__":
    main()
```

When Python executes a source file as the main program, the module's `__name__` is set to `"__main__"`. When the same module is imported, its `__name__` is the module's import name instead.

This makes the following distinction possible:

```text
execute this file directly
        vs.
import this module from another program
```

### Why this matters for DevOps

It lets you keep reusable functions separate from the code that starts the automation.

For example:

```python
def check_service(name):
    return name != ""


def main():
    if check_service("api"):
        print("Service name is valid")


if __name__ == "__main__":
    main()
```

The function can be reused by tests or another module without automatically running `main()` during import.

---

## 4. Passing command-line arguments

Python's command-line interface supports passing arguments to a program.

For example, a program can receive a service name from the command line rather than hard-coding it in the source code.

The `sys` module exposes command-line arguments through `sys.argv`.

```python
import sys

print(sys.argv)
```

If a program is invoked with additional arguments, they are available through that list.

For example, the conceptual structure is:

```text
program.py argument1 argument2
        ↓
        sys.argv
```

For simple inspection or very small programs, `sys.argv` can be useful.

For a real CLI with options, help text, defaults, validation, and clearer user errors, prefer the standard-library `argparse` module. That topic is covered separately in this curriculum.

---

## 5. Script execution versus importing

These two actions are different:

```text
execute a file
    ↓
that file is the main program
```

versus:

```text
import a module
    ↓
Python loads the module so its names can be used
```

Example module:

```python
def health_check():
    return True


if __name__ == "__main__":
    print(health_check())
```

If the file is executed directly, the guarded code runs.

If the file is imported, the function becomes available to the importing code without the guarded block being executed as the module's direct-entry code.

This distinction becomes increasingly important as automation grows from one script into a maintainable project.

---

## 6. Modules and the execution model

Python source files can act as modules.

Suppose a project contains:

```text
automation/
├── main.py
└── checks.py
```

`checks.py`:

```python
def check_service():
    return True
```

`main.py`:

```python
from checks import check_service


def main():
    if check_service():
        print("Service is healthy")


if __name__ == "__main__":
    main()
```

The importing program can use `check_service()` without treating `checks.py` as the program's main entry point.

Import behavior, module search paths, packages, and import-time side effects are covered in the **Modules, Packages, and Imports** topic in the Complete Python track.

---

## 7. Interactive mode versus automation code

Interactive mode is excellent for exploration:

```text
question → experiment → observe → adjust
```

Automation code needs a stronger lifecycle:

```text
input
  ↓
validate
  ↓
perform action
  ↓
check result
  ↓
report/log
```

### Good use of interactive mode

```python
from pathlib import Path

Path("config.yaml").exists()
```

You can quickly inspect the result while learning or troubleshooting.

### Good use of a script

```python
from pathlib import Path


def require_config(path):
    if not path.exists():
        raise FileNotFoundError(path)


def main():
    config = Path("config.yaml")
    require_config(config)
    print(f"Using {config}")


if __name__ == "__main__":
    main()
```

The second form is easier to review, test, reuse, and place under version control.

---

## 8. Running code reproducibly

A DevOps script should not depend on hidden assumptions about where or how it is started.

Before considering an automation script reliable, identify:

- which Python environment it expects;
- which inputs it requires;
- which files or directories it accesses;
- which environment variables it expects;
- which external dependencies it needs;
- what success means;
- what failure should look like.

These concerns are repository and deployment design decisions. They are not automatically guaranteed by Python itself.

For dependency isolation, use Python's virtual-environment and packaging mechanisms as appropriate; those topics are covered later in this curriculum.

---

## 9. Common mistakes

### Mistake 1: Treating the interactive shell as the automation

Interactive commands are useful experiments, but production automation should normally be captured in reviewable source code.

### Mistake 2: Putting all work at module import time

If importing a module immediately performs an operational action, another program or test may trigger that action simply by importing it.

Keep execution boundaries explicit.

### Mistake 3: Hard-coding operational inputs

Avoid embedding environment-specific values directly in reusable automation when they should instead be supplied through configuration or command-line inputs.

### Mistake 4: Ignoring the execution environment

A script can behave differently when launched from a different working directory, interpreter environment, or deployment context. Make important assumptions explicit and test them.

### Mistake 5: Building a complex CLI with `sys.argv` alone

Use `argparse` when the program needs structured command-line options and help behavior.

---

## 10. DevOps exercises

### Exercise 1 — Interactive exploration

In the interactive interpreter:

1. create a string containing an environment name;
2. inspect its type;
3. call one string method;
4. create a list of three services;
5. inspect the list length.

Do not copy the solution from the repository. Type the expressions yourself.

### Exercise 2 — First automation script

Create `service_check.py` that:

1. defines a `check_service()` function;
2. defines a `main()` function;
3. calls `main()` through the `__name__` guard;
4. prints a clear result.

Then import the module from another Python program and verify that importing it does not automatically run the main program.

### Exercise 3 — Command-line input

Create a small program that prints the command-line arguments it receives.

Then change it so that the program expects a service name and reports the supplied value.

After that, rebuild the interface using `argparse` rather than manually interpreting `sys.argv`.

### Exercise 4 — Execution boundary

Create a module containing:

- one reusable function;
- one `main()` function;
- one `if __name__ == "__main__":` guard.

Import the module and execute it directly. Observe which code runs in each case.

---

## 11. Knowledge check

You should be able to answer these without looking at the notes:

1. What is the difference between interactive mode and script execution?
2. What does `__name__` represent in a Python module?
3. Why is `if __name__ == "__main__":` useful?
4. Where can command-line arguments be accessed directly?
5. When is `argparse` preferable to manually processing `sys.argv`?
6. Why can import-time side effects be dangerous in automation?
7. Why should operational assumptions about an execution environment be made explicit?

---

## 12. Completion criteria

You have completed this topic when you can:

- start and use Python's interactive interpreter;
- execute a `.py` source file;
- explain the difference between direct execution and importing;
- use the `__name__` main-program guard correctly;
- inspect command-line arguments with `sys.argv`;
- explain when a CLI should use `argparse`;
- structure a small automation script around a clear entry point;
- identify assumptions that could make execution environment-dependent.

---

## 13. Source notes

### Verified facts

The Python documentation used for this topic establishes the interpreter's interactive mode, script execution model, command-line argument handling, and module execution behavior.

### Repository design decisions

The DevOps examples in this file deliberately emphasize explicit entry points, reusable functions, reproducibility, and operational failure boundaries. These are learning and repository design choices, not claims that Python itself requires this exact project structure.

### Authoritative references

- Python Tutorial — Using the Python Interpreter: https://docs.python.org/3/tutorial/interpreter.html
- Python Command Line and Environment: https://docs.python.org/3/using/cmdline.html
- Python Tutorial — Modules: https://docs.python.org/3/tutorial/modules.html
- Python Standard Library — `sys`: https://docs.python.org/3/library/sys.html
- Python Standard Library — `argparse`: https://docs.python.org/3/library/argparse.html
