# Processes and Exit Codes

DevOps automation frequently starts another program, waits for it, reads its output, and decides whether the workflow succeeded. Python's `subprocess` module provides the standard-library interface for these tasks.

> **Verified fact:** `subprocess.run()` is the recommended approach for invoking subprocesses for use cases it can handle. It waits for the command to complete and returns a `CompletedProcess` object.
>
> **Repository design decision:** This chapter focuses on process control patterns that are useful in scripts, CI/CD jobs, and operational automation.

## 1. Mental model

A subprocess has a lifecycle:

```text
Python automation
      |
      | start
      v
  child process
      |
      +---- stdout
      +---- stderr
      |
      v
  exit status
      |
      v
Python decides: success / failure / retry / report
```

The important questions are:

1. What command should run?
2. What arguments should it receive?
3. Should output be displayed, captured, or redirected?
4. What exit status means success?
5. What should happen if the process fails or times out?

## 2. Run a command

For straightforward process execution:

```python
import subprocess

result = subprocess.run(["python", "health_check.py"])
print(result.returncode)
```

> **Verified fact:** `subprocess.run()` waits for the process to complete and returns a `CompletedProcess` instance.

Using a sequence of arguments makes the command and its arguments explicit:

```python
subprocess.run(["python", "backup.py", "--environment", "production"])
```

The command and arguments here are examples; use commands that actually exist in the target environment.

## 3. Exit codes

A process communicates an exit status when it finishes.

```python
import subprocess

result = subprocess.run(["python", "health_check.py"])

if result.returncode == 0:
    print("command succeeded")
else:
    print(f"command failed: {result.returncode}")
```

> **Verified fact:** Python documents that an exit status of `0` typically indicates successful execution. On POSIX systems, a negative `returncode` indicates termination by signal `N`.

Do not assume that every external program uses the same non-zero code for every failure. Consult that program's official documentation when the distinction matters.

## 4. Fail fast with `check=True`

If a non-zero exit status should make the Python operation fail, `check=True` is often clearer:

```python
import subprocess

subprocess.run(
    ["python", "deploy.py"],
    check=True,
)
```

If the process exits with a non-zero status, Python raises `subprocess.CalledProcessError`.

```python
import subprocess

try:
    subprocess.run(["python", "deploy.py"], check=True)
except subprocess.CalledProcessError as exc:
    print(f"deployment command failed: {exc.returncode}")
    raise
```

> **Verified fact:** With `check=True`, `subprocess.run()` raises `CalledProcessError` when the process exits with a non-zero status.

## 5. Capture standard output

Automation often needs to inspect command output instead of printing it directly.

```python
import subprocess

result = subprocess.run(
    ["python", "status.py"],
    capture_output=True,
    text=True,
    check=True,
)

print(result.stdout)
```

> **Verified fact:** `capture_output=True` captures stdout and stderr. With `text=True`, the captured streams are text rather than bytes.

You can inspect both streams:

```python
print("stdout:", result.stdout)
print("stderr:", result.stderr)
```

Be careful when captured output may contain credentials, tokens, or other sensitive information.

## 6. Standard output and standard error

A child process normally has separate standard output and standard error streams.

```text
child process
   |
   +---- stdout → normal output
   |
   +---- stderr → diagnostic/error output
```

Python lets you explicitly control these streams:

```python
import subprocess

result = subprocess.run(
    ["python", "check.py"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
    text=True,
)
```

`capture_output=True` is a convenient way to request both streams when that is all you need.

## 7. Combine stderr with stdout

When a workflow needs one combined stream, `stderr=subprocess.STDOUT` can redirect standard error into standard output:

```python
import subprocess

result = subprocess.run(
    ["python", "check.py"],
    stdout=subprocess.PIPE,
    stderr=subprocess.STDOUT,
    text=True,
)

print(result.stdout)
```

> **Verified fact:** When stderr is redirected to `STDOUT`, Python documents that the combined stream is available through `stdout`, while `stderr` is `None`.

## 8. Handle expected command failures

Not every non-zero status needs to crash the entire automation program. Sometimes the caller needs to inspect the status and make a decision.

```python
import subprocess

result = subprocess.run(
    ["python", "check.py"],
    capture_output=True,
    text=True,
)

if result.returncode == 0:
    print("check passed")
elif result.returncode == 2:
    print("check reported a known condition")
else:
    raise RuntimeError(
        f"unexpected exit status: {result.returncode}"
    )
```

The meaning of `2` in this example is repository demonstration only. A real command's exit-code meanings must come from its authoritative documentation.

## 9. Timeouts

A subprocess should not necessarily be allowed to wait forever.

```python
import subprocess

try:
    subprocess.run(
        ["python", "health_check.py"],
        timeout=30,
        check=True,
    )
except subprocess.TimeoutExpired:
    print("command timed out")
    raise
```

> **Verified fact:** If the timeout expires while `subprocess.run()` is waiting for the child process, Python kills and waits for the child before re-raising `TimeoutExpired`.

For the broader retry and timeout design, see [Retries and Timeouts](./Retries%20and%20Timeouts.md).

## 10. Environment variables

A child process normally inherits the current process environment. Python can instead receive an explicit environment mapping through `env`.

```python
import os
import subprocess

env = os.environ.copy()
env["APP_MODE"] = "production"

subprocess.run(
    ["python", "worker.py"],
    env=env,
    check=True,
)
```

> **Verified fact:** When `env` is provided, Python uses that mapping for the new process instead of the default environment inheritance behavior.

Avoid putting secrets directly into source code. Use the environment carefully and avoid printing sensitive values.

## 11. Working directory

A command may need to run from a particular directory.

```python
import subprocess

subprocess.run(
    ["python", "build.py"],
    cwd="project",
    check=True,
)
```

`cwd` changes the working directory for the child process. Validate paths and make the working-directory assumption explicit in automation.

## 12. Avoid `shell=True` unless it is required

For ordinary command execution, passing an argument sequence is usually clearer than constructing a shell command string.

Prefer:

```python
subprocess.run(
    ["python", "deploy.py", "--environment", "production"],
    check=True,
)
```

Be especially careful when incorporating external input into shell commands. Shell interpretation can change the meaning of input and create security risks.

> **Repository design decision:** Examples in this chapter use argument sequences and do not rely on `shell=True` for ordinary command execution.

If a task genuinely requires shell features, verify the exact behavior and security implications for the target platform and shell.

## 13. Preserve useful failure information

When a command fails, useful diagnostics can include:

- Command arguments
- Exit status
- Captured stderr
- Captured stdout when relevant
- Timeout information
- The surrounding operation being attempted

Example:

```python
import subprocess

try:
    result = subprocess.run(
        ["python", "build.py"],
        capture_output=True,
        text=True,
        check=True,
    )
except subprocess.CalledProcessError as exc:
    print(f"command failed with status {exc.returncode}")
    print(exc.stderr)
    raise
```

Do not automatically print every captured stream in production. First consider whether it can contain secrets or excessive output.

## 14. Process return status is not application truth

An exit code is one signal from a child process. A command can exit successfully while the larger automation goal still needs verification.

For example:

```text
command exited 0
      ↓
process-level success
      ↓
verify desired operational state
      ↓
workflow success
```

> **Repository design decision:** Treat exit status as an important signal, but perform explicit post-command verification when the automation's actual goal requires it.

This is especially useful for deployment and infrastructure workflows where “the command ran successfully” and “the desired state was reached” are not necessarily the same question.

## 15. Process execution and idempotency

The previous topics connect here:

```text
process execution
      ↓
exit status
      ↓
retry?
      ↓
idempotency analysis
      ↓
verify desired state
```

If a command partially changes state and then returns an uncertain result, blindly rerunning it may create unintended effects. Understand the command's behavior before adding retries.

## 16. Practical command wrapper

A small wrapper can centralize common process-handling behavior:

```python
import subprocess


def run_command(command: list[str]) -> str:
    result = subprocess.run(
        command,
        capture_output=True,
        text=True,
        check=True,
    )
    return result.stdout


output = run_command(["python", "status.py"])
print(output)
```

The type annotation here is a repository example. It assumes the caller supplies a list of strings representing the command and arguments.

A production wrapper may need additional policy for timeouts, logging, environment handling, and error classification.

## 17. Common mistakes

### Mistake 1: Ignoring the exit status

A script launches a command and continues as if it succeeded.

**Better:** inspect `returncode` or use `check=True` when failure should propagate.

### Mistake 2: Assuming every non-zero code means the same thing

Different programs define their own exit-status conventions.

**Better:** consult the command's official documentation.

### Mistake 3: Capturing everything without limits

Large output can consume memory or make diagnostics unwieldy.

**Better:** choose output handling deliberately for the command and workflow.

### Mistake 4: Printing secrets

Captured stdout/stderr can contain sensitive values.

**Better:** review what gets logged or displayed.

### Mistake 5: Building shell commands from untrusted input

Shell interpretation can create injection risks.

**Better:** prefer argument sequences and avoid unnecessary shell interpretation.

### Mistake 6: Retrying a failed command blindly

The first attempt may have changed state before failing.

**Better:** analyze idempotency and the command's failure semantics first.

### Mistake 7: Treating exit code zero as complete verification

The process may succeed while the desired external state is still wrong.

**Better:** perform an explicit verification step when the workflow requires it.

## 18. Practical DevOps exercise

Build a Python deployment-command wrapper.

Requirements:

1. Accept a command and its arguments from a controlled configuration.
2. Execute it without unnecessary shell interpretation.
3. Capture stdout and stderr.
4. Apply a finite timeout.
5. Record the exit status.
6. Report useful failure information without exposing secrets.
7. Preserve a non-zero failure for the calling workflow.
8. Add a post-command verification step.

### Extension
Add a small policy layer that classifies failures into:

```text
success
known/expected condition
retryable failure
fatal failure
```

Use the actual command's documented exit statuses rather than the demonstration values in this chapter.

## 19. Knowledge checks

1. What does `subprocess.run()` return?
2. What does `returncode == 0` typically indicate?
3. What does `check=True` change?
4. How can stdout and stderr be captured?
5. What does `TimeoutExpired` represent?
6. Why should shell interpretation be avoided when it is unnecessary?
7. How can a child process receive a custom environment?
8. Why is exit status not always sufficient to verify the desired operational state?
9. Why should idempotency be considered before retrying a failed command?
10. Which exit-code meanings must be verified from the target command's documentation?

## 20. Key takeaways

- `subprocess.run()` is the standard starting point for straightforward subprocess execution.
- Use explicit argument sequences for ordinary command invocation.
- Inspect or enforce exit status deliberately.
- Use `check=True` when a non-zero exit should become a Python exception.
- Capture stdout/stderr only when the workflow needs them and handle sensitive output carefully.
- Set timeouts for operations that should not wait indefinitely.
- Pass environment and working-directory requirements explicitly when needed.
- Avoid unnecessary shell interpretation.
- Treat process exit status as one signal; verify external state when required.
- Analyze idempotency before retrying state-changing commands.

## Verified facts vs repository design

### Verified facts

- Python's `subprocess` module is intended to spawn processes, connect to their input/output/error pipes, and obtain return codes.
- Python recommends `subprocess.run()` for use cases it can handle.
- `subprocess.run()` returns a `CompletedProcess` object after the command completes.
- `check=True` raises `CalledProcessError` for a non-zero exit status.
- `capture_output=True` captures stdout and stderr.
- `text=True` makes standard streams text rather than bytes.
- `env` provides the environment mapping for the new process.
- `cwd` can specify the child's working directory.
- `timeout` causes `TimeoutExpired` when the wait exceeds the specified timeout.

### Repository design decisions

- Examples prefer argument sequences over unnecessary shell interpretation.
- Process success is treated separately from verification of the desired operational state.
- Retry decisions are connected to idempotency and command-specific semantics.

### Demonstrations

- Example commands such as `health_check.py`, `deploy.py`, and `status.py` are illustrative names, not claims that these files exist in the repository.
- The exit status `2` used in one example is intentionally illustrative; its meaning must be verified for a real command.

## Authoritative references

- Python `subprocess`: https://docs.python.org/3/library/subprocess.html
- Python `os`: https://docs.python.org/3/library/os.html
- Python standard library: https://docs.python.org/3/library/
