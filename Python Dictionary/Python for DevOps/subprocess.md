# `subprocess`

The `subprocess` module lets Python start child processes, connect to their standard input/output/error streams, and inspect their return codes.

> **Source note:** The API and behavior described here are based on the official Python documentation for `subprocess`. The examples are teaching demonstrations created for this repository.

---

## 1. The mental model

Think of a subprocess call as a boundary between Python and another program:

```text
Python
  ↓
start child process
  ↓
provide arguments / input
  ↓
wait for completion
  ↓
inspect stdout / stderr / return code
  ↓
handle success or failure
```

This is particularly useful in DevOps automation because Python often needs to coordinate existing command-line tools.

The important distinction is that starting a command is not the same as proving that the command succeeded.

---

## 2. Start with `subprocess.run()`

The official documentation recommends `subprocess.run()` for the use cases it can handle. More advanced process-management requirements can use `Popen` directly.

A basic example:

```python
import subprocess

result = subprocess.run(["python", "--version"])

print(result.returncode)
```

`run()` waits for the command to complete and returns a `CompletedProcess` object.

For portability when launching the current Python interpreter, the official documentation recommends `sys.executable`:

```python
import subprocess
import sys

subprocess.run([sys.executable, "script.py"])
```

---

## 3. Pass arguments as a sequence

For normal command execution, prefer a sequence of arguments:

```python
subprocess.run([
    "python",
    "script.py",
    "--environment",
    "staging",
])
```

The official documentation states that providing a sequence is generally preferred because the module can handle the required escaping and quoting of arguments.

This is especially useful when an argument contains spaces:

```python
subprocess.run([
    "tool",
    "--file",
    "path with spaces/config.json",
])
```

Do not manually build a shell command string when a structured argument sequence is sufficient.

---

## 4. `CompletedProcess`

`subprocess.run()` returns a `CompletedProcess` instance.

Important attributes include:

```python
result.args
result.returncode
result.stdout
result.stderr
```

For example:

```python
result = subprocess.run(["python", "--version"])

print(result.args)
print(result.returncode)
```

A return code of `0` typically indicates successful execution. The exact meaning of other codes is determined by the child program and its environment.

On POSIX systems, a negative return code can indicate termination by a signal.

---

## 5. Capture standard output

By default, `subprocess.run()` does not capture the child process's standard output for you.

Use `capture_output=True` when you need to inspect stdout and stderr:

```python
import subprocess

result = subprocess.run(
    ["python", "--version"],
    capture_output=True,
    text=True,
)

print(result.stdout)
print(result.stderr)
```

With `capture_output=True`, stdout and stderr are captured.

When `text=True` is used, the streams are handled as text rather than bytes.

---

## 6. `stdout`, `stderr`, and `PIPE`

The lower-level API can explicitly connect streams using `PIPE`:

```python
import subprocess

result = subprocess.run(
    ["tool", "--check"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
    text=True,
)
```

`capture_output=True` is a convenient way to request both stdout and stderr capture.

If you want stderr combined with stdout, the documentation describes using:

```python
stderr=subprocess.STDOUT
```

instead of `capture_output=True`.

---

## 7. Treat the return code as data

A command can start successfully and still report an operational failure.

Prefer checking the result explicitly when non-zero exit codes are expected to be handled by your application:

```python
result = subprocess.run(
    ["tool", "--check"],
    capture_output=True,
    text=True,
)

if result.returncode != 0:
    print(result.stderr)
```

This is a repository design principle: automation should make the child process's outcome observable and actionable.

---

## 8. Fail automatically with `check=True`

If a non-zero exit status should be treated as an exception, use `check=True`:

```python
import subprocess

subprocess.run(
    ["tool", "--check"],
    check=True,
)
```

If the process exits with a non-zero status, `subprocess.run()` raises `subprocess.CalledProcessError`.

This is useful when failure should stop the current operation unless explicitly handled.

---

## 9. Handle `CalledProcessError`

You can handle a failed command explicitly:

```python
import subprocess

try:
    subprocess.run(
        ["tool", "--check"],
        check=True,
        capture_output=True,
        text=True,
    )
except subprocess.CalledProcessError as exc:
    print(f"command failed with exit code {exc.returncode}")
    print(exc.stderr)
```

When output was captured, `CalledProcessError` can expose the command, return code, and captured output through its documented attributes.

Avoid catching every exception and treating the command as successful.

---

## 10. Use timeouts for operations that must not wait forever

`subprocess.run()` accepts a `timeout` value in seconds:

```python
import subprocess

subprocess.run(
    ["tool", "--check"],
    timeout=30,
)
```

If the timeout expires, `subprocess.TimeoutExpired` is raised after the child process has been terminated and waited for according to the documented behavior of `run()`.

A timeout is particularly valuable for automation that must have a bounded execution time.

```python
try:
    subprocess.run(
        ["tool", "--check"],
        timeout=30,
        check=True,
    )
except subprocess.TimeoutExpired:
    print("command timed out")
```

The timeout applies to waiting for the process; the documentation notes that process creation itself cannot necessarily be interrupted immediately on every platform.

---

## 11. Text versus bytes

Without text mode or an explicit encoding, subprocess streams are opened as binary streams.

For text output:

```python
result = subprocess.run(
    ["tool", "--version"],
    capture_output=True,
    text=True,
)
```

You can also specify an encoding explicitly when appropriate:

```python
result = subprocess.run(
    ["tool", "--version"],
    capture_output=True,
    text=True,
    encoding="utf-8",
)
```

Choose the encoding based on the interface you are integrating with rather than assuming every external program uses the same encoding.

---

## 12. Send input to a process

`run()` can provide data to a child's standard input through `input`:

```python
result = subprocess.run(
    ["tool"],
    input="configuration data\n",
    text=True,
    capture_output=True,
)
```

When `input` is supplied, Python connects the child's stdin to a pipe for the operation.

Use this when the child program is explicitly designed to receive input through stdin.

Avoid passing sensitive material through subprocess input unless the surrounding process and child program provide an appropriate security boundary.

---

## 13. Working directory

Use `cwd` when the child process should execute with a particular working directory:

```python
from pathlib import Path
import subprocess

project_dir = Path("project")

subprocess.run(
    ["tool", "build"],
    cwd=project_dir,
    check=True,
)
```

This is often cleaner than changing Python's own working directory before launching the child.

The exact behavior of executable lookup can depend on the platform and on `cwd`/`env`, so use a fully qualified executable path when maximum reliability is required.

---

## 14. Environment variables

Use `env` when the child process needs a specific environment mapping:

```python
import os
import subprocess

child_env = os.environ.copy()
child_env["APP_ENV"] = "staging"

subprocess.run(
    ["tool", "status"],
    env=child_env,
    check=True,
)
```

When `env` is provided, it defines the environment for the new process rather than using the default inherited environment behavior.

Copying the existing environment before changing one value is a common repository design pattern when the child should retain the rest of the environment.

---

## 15. `shell=True` is a deliberate choice

By default, `subprocess` does not execute the command through a shell.

`shell=True` requests shell execution:

```python
subprocess.run(
    "echo hello",
    shell=True,
)
```

Shell execution can be useful when shell-specific features are actually required, such as shell pipes or wildcard expansion.

However, the official documentation specifically directs users to its security considerations before using `shell=True`.

Do not pass untrusted input into a shell command without understanding the resulting command-injection risk.

For ordinary program execution, prefer a sequence of arguments with the default `shell=False`.

---

## 16. Do not confuse Python and shell features

Some operations commonly performed in shells have Python equivalents.

The official documentation points to facilities such as:

- `glob`
- `fnmatch`
- `os.walk()`
- `os.path.expandvars()`
- `os.path.expanduser()`

For example, if the purpose is simply to find files, invoking a shell only to perform wildcard expansion may add unnecessary complexity.

Prefer the Python API when it gives you the control and portability you need.

---

## 17. Pipelines and advanced process management

A pipeline such as:

```text
command-a | command-b
```

requires multiple processes and connected streams.

For more advanced process-management requirements, Python provides the `Popen` interface.

A simplified pattern is:

```python
import subprocess

first = subprocess.Popen(
    ["command-a"],
    stdout=subprocess.PIPE,
)

second = subprocess.Popen(
    ["command-b"],
    stdin=first.stdout,
)

first.stdout.close()
second.wait()
```

This is only a conceptual example. Real pipelines require careful handling of process lifetime, pipes, errors, and output consumption.

Use `run()` unless you genuinely need the additional control provided by `Popen`.

---

## 18. `Popen` mental model

`Popen` represents a lower-level child-process interface.

Conceptually:

```text
Popen(...)
   ↓
child process exists
   ↓
communicate / wait / inspect
   ↓
process completion
```

The official documentation describes `Popen` as the underlying interface for more advanced use cases.

Important areas include:

- process creation
- stdin/stdout/stderr pipes
- environment
- working directory
- process waiting
- communication
- process lifecycle

Do not choose `Popen` simply because it looks more powerful. Additional control also creates additional lifecycle responsibilities.

---

## 19. Avoid deadlocks when using pipes

When working directly with `Popen`, process I/O must be handled carefully.

The documented `communicate()` method is designed to interact with the process through its pipes and wait for completion.

For example:

```python
process = subprocess.Popen(
    ["tool"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
    text=True,
)

stdout, stderr = process.communicate()
```

Do not casually read one pipe to completion while ignoring another pipe that the child may be filling. For complex process I/O, follow the official `Popen` and `communicate()` documentation rather than inventing a custom pipe-management strategy.

---

## 20. Executable selection and portability

Executable lookup is platform-dependent.

The official documentation recommends a fully qualified executable path when maximum reliability is required.

When intentionally searching `PATH`, Python provides `shutil.which()`:

```python
import shutil

path = shutil.which("tool")

if path is None:
    raise RuntimeError("tool is not available")
```

For launching the current Python interpreter, use `sys.executable` rather than assuming the command name is `python`.

This matters when automation runs inside virtual environments or on systems where the Python executable has a different name.

---

## 21. Command execution as an automation boundary

A robust DevOps wrapper should make the subprocess contract explicit:

```text
command
  ↓
arguments
  ↓
working directory / environment
  ↓
execution
  ↓
stdout / stderr / return code
  ↓
validation
  ↓
success or failure
```

For example:

```python
import subprocess


def run_check(command):
    result = subprocess.run(
        command,
        capture_output=True,
        text=True,
        timeout=30,
    )

    if result.returncode != 0:
        raise RuntimeError(
            f"command failed: {result.returncode}\n{result.stderr}"
        )

    return result.stdout
```

This is a repository teaching pattern, not a universal error-handling policy. Production wrappers should define how much command output may be retained, logged, or exposed.

---

## 22. Do not log secrets accidentally

Subprocess automation can expose sensitive data through:

- command arguments
- environment variables
- stdin
- stdout
- stderr
- exception messages

Avoid blindly logging the complete command, environment, or captured output when sensitive values may be present.

A safe logging policy should be designed around the specific command being executed.

---

## 23. Common mistakes

### Mistake 1: assuming command start means success

Always define how success is determined. The return code is part of the subprocess result.

### Mistake 2: ignoring non-zero exit codes

Use explicit `returncode` handling or `check=True` when failure should become an exception.

### Mistake 3: using `shell=True` unnecessarily

Prefer structured arguments and the default shell behavior unless shell features are actually required.

### Mistake 4: manually constructing quoted command strings

Pass arguments as a sequence when possible.

### Mistake 5: waiting forever

Use a timeout when the automation requires bounded execution.

### Mistake 6: assuming output is text

Subprocess streams are binary by default. Use text mode or an explicit encoding when appropriate.

### Mistake 7: leaking command output

Captured stdout/stderr may contain credentials, tokens, paths, or other sensitive operational data.

### Mistake 8: using `Popen` for simple commands

Prefer `run()` when it provides everything the task needs.

### Mistake 9: inventing exit-code meanings

The meaning of a non-zero code is defined by the program being executed. Do not claim a universal meaning for a tool-specific code without verification.

---

## 24. DevOps practice exercise

Build a Python wrapper around a harmless local command available in your environment.

Requirements:

1. Pass the executable and arguments as a sequence.
2. Capture stdout and stderr.
3. Use text mode.
4. Apply a timeout.
5. Inspect the return code.
6. Handle a non-zero result explicitly.
7. Do not use `shell=True` unless the exercise specifically requires a shell feature.
8. Avoid logging sensitive environment values or command output unnecessarily.
9. Make the executable configurable without hard-coding an environment-specific path.
10. Write a test that exercises both a successful command and a controlled failure.

Then extend the wrapper to accept a working directory and selected environment variables.

---

## 25. Knowledge checks

Before moving on, you should be able to answer:

1. What problem does `subprocess` solve?
2. Why is `subprocess.run()` usually the starting point?
3. What does `CompletedProcess.returncode` represent?
4. What is the difference between checking `returncode` and using `check=True`?
5. What exception is raised for a non-zero exit code with `check=True`?
6. How do you capture stdout and stderr?
7. What does `text=True` change?
8. Why are argument sequences generally preferred over manually built command strings?
9. When would `shell=True` be appropriate?
10. Why can `shell=True` be a security concern with untrusted input?
11. How does `timeout` affect `run()`?
12. What is `Popen` used for?
13. How can a child process receive a custom environment?
14. Why might `sys.executable` be preferable to the literal `python` command?
15. Why should subprocess output be treated carefully in logs?

---

## 26. Completion criteria

You are ready to move on when you can:

- execute a child process with `subprocess.run()`
- pass arguments safely as a sequence
- inspect a `CompletedProcess`
- capture stdout and stderr
- use text mode appropriately
- handle non-zero exit codes
- use `check=True` when appropriate
- catch `CalledProcessError`
- apply execution timeouts
- provide custom environment variables
- choose a working directory
- explain why `shell=True` deserves special care
- explain when `Popen` is justified
- avoid leaking sensitive subprocess information

---

## 27. Verified facts vs repository design

### Verified facts

- Python's `subprocess` module can create child processes, connect to standard streams, and obtain return codes.
- `subprocess.run()` is the recommended high-level interface for use cases it can handle.
- `Popen` provides a lower-level interface for more advanced process management.
- `run()` returns a `CompletedProcess` object.
- `check=True` causes a non-zero exit status to raise `CalledProcessError`.
- `timeout` can cause `TimeoutExpired` when the waiting operation exceeds the configured limit.
- `capture_output=True` captures stdout and stderr.
- `text=True` enables text-mode stream handling.
- `env` can define the environment for the child process.
- A sequence of arguments is generally preferred for normal command execution.
- `shell=True` executes the command through a shell and has documented security considerations.

### Repository design decisions

- Prefer `subprocess.run()` for ordinary DevOps command wrappers.
- Treat the return code as part of the automation contract.
- Keep command execution separate from application logic.
- Use timeouts when the automation requires bounded execution.
- Avoid logging complete command output when it may contain sensitive data.

### Demonstrations

All command examples in this file are teaching demonstrations. Their exact output depends on the installed program, operating system, environment, and command arguments.

---

## 28. Authoritative references

- Python `subprocess` documentation: https://docs.python.org/3/library/subprocess.html
- Python `os` documentation: https://docs.python.org/3/library/os.html
- Python `shutil.which()` documentation: https://docs.python.org/3/library/shutil.html#shutil.which
