# Retries and Timeouts

Reliable automation must handle operations that take too long, fail temporarily, or become uncertain after a request has been sent. **Timeouts** put an upper bound on how long an operation should wait. **Retries** give an operation another chance when retrying is appropriate.

> **Verified fact:** Python's `subprocess.run()` accepts a `timeout` argument and raises `TimeoutExpired` when the timeout expires while waiting for the child process.
>
> **Verified fact:** `urllib.request.urlopen()` accepts a timeout in seconds for blocking operations such as the connection attempt.
>
> **Repository design decision:** This chapter teaches retries as an explicit policy rather than as a generic “retry everything” wrapper.

## 1. Mental model

Think about a remote or external operation as having a bounded execution policy:

```text
attempt
  ↓
completed? ── yes → success
  │
  no
  ↓
was it a timeout/transient failure?
  │
  ├── no → fail
  │
  └── yes
       ↓
   retry budget available?
       │
       ├── no → fail
       └── yes → wait → attempt again
```

A useful policy answers four questions:

1. **What is the timeout?**
2. **Which failures are retryable?**
3. **How many attempts are allowed?**
4. **How long should the next attempt wait?**

Do not hide these decisions inside an unexplained loop.

## 2. Why timeouts matter

Without a timeout, an automation task can wait much longer than the workflow expects. This is especially important when the script is running in CI/CD, a scheduled job, or another automation system with its own execution limits.

For example, `urllib.request.urlopen()` accepts a timeout:

```python
from urllib.request import urlopen

with urlopen("https://example.com", timeout=5) as response:
    body = response.read()
```

The exact timeout should be chosen for the operation and environment. There is no universal value that is correct for every system.

## 3. Subprocess timeouts

Python's `subprocess.run()` also supports a timeout:

```python
import subprocess

result = subprocess.run(
    ["python", "health_check.py"],
    capture_output=True,
    text=True,
    timeout=30,
    check=True,
)
```

If the timeout expires while waiting for the child process, Python raises `subprocess.TimeoutExpired`.

```python
import subprocess

try:
    subprocess.run(["python", "health_check.py"], timeout=30, check=True)
except subprocess.TimeoutExpired:
    print("health check timed out")
```

> **Verified fact:** When `subprocess.run()` times out, the child process is killed and waited for before `TimeoutExpired` is re-raised. Process creation itself cannot be interrupted on many platform APIs, so the timeout is not a guarantee that an exception occurs immediately.

## 4. Retry only what is retryable

A retry is appropriate only when repeating the operation is understood to be safe and the failure may reasonably be temporary.

Potentially retryable situations can include a documented transient network failure or a temporary service-unavailable condition. The exact classification depends on the system being automated.

Do not automatically retry:

- Invalid configuration
- Authentication failures that require operator action
- Invalid input
- Programming errors
- Operations known to have unsafe repeated side effects

The target API or command documentation should define the failure conditions and retry expectations when available.

## 5. Retries and idempotency belong together

Retries can repeat an operation after the client is uncertain about what happened.

For example:

```text
Client → request → server
             ↓
        operation happens
             ↓
       response is lost
             ↓
       client sees failure
             ↓
          retry?
```

The client may not know whether the original operation completed.

This is why the previous **Idempotency** topic matters. Before retrying a state-changing operation, determine whether the operation can safely be repeated or whether the target API provides a documented mechanism for deduplicating retries.

> **Repository design decision:** Treat retry policy and side-effect analysis as one design problem rather than adding retries as a generic final step.

## 6. A simple bounded retry loop

For a small Python automation task, an explicit loop can make the policy easy to read:

```python
import time

MAX_ATTEMPTS = 3
DELAY_SECONDS = 2

for attempt in range(1, MAX_ATTEMPTS + 1):
    try:
        perform_operation()
        break
    except TemporaryError:
        if attempt == MAX_ATTEMPTS:
            raise
        time.sleep(DELAY_SECONDS)
```

This is a teaching pattern. `TemporaryError` and `perform_operation()` are placeholders and must be replaced with real, verified exceptions and operations for a particular system.

The important properties are:

- A finite attempt count
- An explicit retry condition
- A delay between attempts
- The final failure is not silently discarded

## 7. Exponential backoff

Using the same delay for every retry is not always appropriate. A common design is to increase the delay after each failed attempt.

A simple exponential sequence is:

```text
base delay
2 × base delay
4 × base delay
8 × base delay
...
```

For example:

```python
import time

base_delay = 1
max_attempts = 4

for attempt in range(1, max_attempts + 1):
    try:
        perform_operation()
        break
    except TemporaryError:
        if attempt == max_attempts:
            raise

        delay = base_delay * (2 ** (attempt - 1))
        time.sleep(delay)
```

This is a repository demonstration of a backoff calculation, not a universal recommendation for every API.

## 8. Add a maximum delay

An unrestricted exponential calculation can eventually produce an impractical wait.

A bounded policy can cap the delay:

```python
base_delay = 1
max_delay = 30

for attempt in range(1, 5):
    try:
        perform_operation()
        break
    except TemporaryError:
        if attempt == 4:
            raise

        delay = min(max_delay, base_delay * (2 ** (attempt - 1)))
        time.sleep(delay)
```

The maximum delay is a design choice and should reflect the operation's requirements.

## 9. Jitter

If many automation workers retry at the same time using exactly the same schedule, they can synchronize their retries.

A retry policy may introduce **jitter**, meaning a controlled random variation in the delay. The exact jitter algorithm should come from the requirements or the target service's documented guidance.

For example, a repository demonstration can make the delay vary around a calculated backoff value:

```python
import random

backoff = 4
jittered_delay = random.uniform(0, backoff)
```

This is only an illustration of the concept. Do not assume this particular distribution is the correct policy for a production service.

## 10. Use a monotonic clock for elapsed-time decisions

When measuring elapsed time for a local timeout or retry budget, a monotonic clock is generally the appropriate clock because it is not intended to move backward when the system wall clock changes.

Python provides `time.monotonic()`:

```python
import time

start = time.monotonic()

# perform work

elapsed = time.monotonic() - start
print(f"elapsed: {elapsed:.2f}s")
```

> **Verified fact:** Python documents `time.monotonic()` as a monotonic clock that cannot go backward and is not affected by system clock updates.

Do not use wall-clock timestamps as a substitute for elapsed-duration measurement when a monotonic clock is appropriate.

## 11. Timeout versus retry budget

These are different controls.

```text
per-attempt timeout
        ↓
maximum time allowed for one operation

retry limit
        ↓
maximum number of attempts

overall deadline/budget
        ↓
maximum time allowed for the whole workflow
```

For example, three attempts with a 5-second timeout each do not automatically mean the whole workflow lasts exactly 15 seconds. Connection setup, processing, delays, and other overhead can affect the total duration.

> **Repository design decision:** For production automation, document whether a timeout is per operation, per attempt, or an overall workflow budget.

## 12. HTTP timeout example

Python's standard library can specify a timeout for `urllib.request.urlopen()`:

```python
from urllib.error import URLError
from urllib.request import urlopen

try:
    with urlopen("https://example.com/health", timeout=5) as response:
        body = response.read()
except URLError as exc:
    print(f"request failed: {exc}")
```

A timeout is not the same thing as a retry policy. If retrying is required, the code must separately decide whether the particular failure is retryable and whether repeating the request is safe.

Python's documentation also notes that `urllib.request` is a lower-level interface and recommends the Requests package for a higher-level HTTP client interface.

## 13. Preserve the final failure

A dangerous retry loop is one that keeps retrying and eventually reports success even though the operation never succeeded.

Prefer a clear failure path:

```python
attempts = 3

for attempt in range(attempts):
    try:
        perform_operation()
        break
    except TemporaryError:
        if attempt == attempts - 1:
            raise
```

The final exception should remain visible to the caller, logging system, CI job, or orchestrator.

## 14. Logging retry behavior

Retrying without visibility can make incidents difficult to diagnose.

A useful log record can identify:

- Operation being attempted
- Attempt number
- Maximum attempts
- Reason for retry
- Planned delay
- Final failure

Avoid logging credentials, access tokens, or other secrets while adding retry diagnostics.

Example:

```python
import logging

logger = logging.getLogger(__name__)

logger.warning(
    "operation failed; retrying",
    extra={"attempt": attempt, "max_attempts": max_attempts},
)
```

The exact logging structure is a repository design choice.

## 15. Common mistakes

### Mistake 1: Retry everything

Not every exception represents a temporary failure.

**Better:** explicitly classify retryable failures.

### Mistake 2: Retry without a limit

An automation job can remain stuck indefinitely.

**Better:** define a finite attempt limit and/or overall deadline.

### Mistake 3: Retry unsafe operations

A repeated request can create duplicate or unintended state.

**Better:** analyze idempotency and target-system semantics first.

### Mistake 4: Use only a large timeout

A huge timeout can make a failure slow to surface.

**Better:** choose a timeout based on the operation's expected behavior and workflow requirements.

### Mistake 5: Hide the final exception

A job can appear successful even though all attempts failed.

**Better:** propagate or explicitly report the final failure.

### Mistake 6: Retry at every layer

A client, library, service, and orchestrator can all retry the same operation, multiplying attempts unexpectedly.

**Better:** understand which layer owns retries and document the boundary.

### Mistake 7: Use wall-clock time for elapsed deadlines

System clock changes can make elapsed-time calculations unreliable.

**Better:** use a monotonic clock for elapsed-duration measurements.

## 16. Practical DevOps exercise

Build a Python health-check utility with a bounded retry policy.

Requirements:

1. Accept a target URL from the command line.
2. Apply a per-attempt timeout.
3. Attempt the health check a finite number of times.
4. Wait between failed attempts.
5. Log the attempt number and final failure.
6. Exit with a non-zero status if all attempts fail.
7. Ensure the retry policy does not log secrets.
8. Test a successful first attempt.
9. Test repeated temporary failures.
10. Test a timeout.

### Extension
Add exponential backoff and a maximum delay. Then write a test that verifies the calculated delays without actually sleeping for the full duration.

## 17. Knowledge checks

1. What problem does a timeout solve?
2. Why should retries be limited?
3. What makes a failure potentially retryable?
4. Why should idempotency be considered before retrying a state-changing operation?
5. What is exponential backoff?
6. Why can jitter be useful when many workers retry simultaneously?
7. Why is `time.monotonic()` useful for elapsed-time measurement?
8. What is the difference between a per-attempt timeout and an overall retry budget?
9. Why should the final failure remain visible?
10. Where should retry ownership be documented when multiple layers can retry?

## 18. Key takeaways

- **Timeouts bound waiting; retries repeat an operation under a defined policy.**
- Retry only failures that are understood to be temporary and safe to repeat.
- Keep attempts finite and delays explicit.
- Consider idempotency before retrying state-changing operations.
- Use backoff when the retry schedule should spread attempts over time.
- Consider jitter when synchronized retries could be a problem.
- Use `time.monotonic()` for elapsed-time decisions.
- Preserve and report the final failure.
- Verify retry behavior from the target system's official documentation.

## Verified facts vs repository design

### Verified facts

- Python `subprocess.run()` supports `timeout` and raises `TimeoutExpired` when the timeout expires while waiting for the child process.
- Python `urllib.request.urlopen()` accepts a timeout in seconds for blocking operations such as connection attempts.
- Python provides `time.monotonic()` as a monotonic clock suitable for measuring elapsed durations.
- Python's standard library documents `time.sleep()` for suspending execution for a specified number of seconds.

### Repository design decisions

- Retry policies in this chapter are explicit and bounded.
- Retryability is treated as an operation-specific decision rather than a universal exception rule.
- Exponential backoff and jitter examples are presented as demonstrations rather than universal production policies.
- The health-check exercise uses a finite retry policy and explicit failure reporting.

### Demonstrations

- `perform_operation()` and `TemporaryError` are placeholders used to explain retry structure. They are not Python standard-library names.
- The backoff and jitter snippets demonstrate policy calculations; they do not establish a service-specific retry contract.

## Authoritative references

- Python `time`: https://docs.python.org/3/library/time.html
- Python `subprocess`: https://docs.python.org/3/library/subprocess.html
- Python `urllib.request`: https://docs.python.org/3/library/urllib.request.html
- Python `socket`: https://docs.python.org/3/library/socket.html
- Python standard library: https://docs.python.org/3/library/
