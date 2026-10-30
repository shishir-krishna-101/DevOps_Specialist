# `logging`

Python's `logging` module provides a standard event-logging system for applications and libraries. For DevOps automation, logging turns important execution events into information that can be inspected during normal operation, troubleshooting, and incident investigation.

> **Source note:** The API and behavior described here are based on the official Python documentation for `logging`. The examples and engineering patterns are teaching demonstrations created for this repository.

---

## 1. Logging mental model

Think of logging as an event pipeline:

```text
application code
      ↓
    Logger
      ↓
  LogRecord
      ↓
  Handler(s)
      ↓
 Formatter
      ↓
console / file / other destination
```

The standard library separates these responsibilities:

- **Logger** — the interface application code uses to record events.
- **Handler** — sends log records to a destination.
- **Formatter** — defines how records are represented.
- **Filter** — provides finer-grained control over which records are handled.

This separation is useful in automation because the code that reports an event does not have to know every detail about where logs will ultimately be written.

---

## 2. Logging versus `print()`

`print()` is useful for ordinary console output from a command-line program.

Logging is useful when the event itself matters to operation, diagnosis, or monitoring.

A practical distinction is:

```text
print()   → ordinary user-facing program output
logging   → operational events and diagnostics
```

The Python Logging HOWTO specifically recommends logging for events that occur during normal program operation and for fault investigation.

For example:

```python
print("Deployment complete")
```

is ordinary program output, while:

```python
logger.info("deployment completed")
```

records an operational event through the logging system.

The correct choice depends on what the output is intended to communicate.

---

## 3. Create a module-level logger

The recommended pattern for application modules is:

```python
import logging

logger = logging.getLogger(__name__)
```

Then use the logger:

```python
logger.debug("starting validation")
logger.info("configuration loaded")
logger.warning("optional configuration is missing")
logger.error("deployment failed")
```

Using `__name__` gives the logger a name that follows the module's package namespace.

This also participates naturally in Python's hierarchical logging system.

---

## 4. Logging levels

Python defines standard severity levels. In increasing order of severity:

| Level | Typical purpose |
|---|---|
| `DEBUG` | Detailed diagnostic information. |
| `INFO` | Confirmation that normal operation is proceeding as expected. |
| `WARNING` | Something unexpected happened, or a condition may require attention, while the program can continue. |
| `ERROR` | A more serious problem prevented some operation from completing. |
| `CRITICAL` | A serious problem indicating the program may be unable to continue. |

Example:

```python
logger.debug("parsed 42 records")
logger.info("deployment started")
logger.warning("retrying after transient failure")
logger.error("deployment command failed")
logger.critical("required service is unavailable")
```

The severity should describe the event rather than simply how dramatic the message sounds.

---

## 5. The default threshold

The root logger starts with a default level of `WARNING`.

Consequently, without additional configuration, lower-severity events such as `DEBUG` and `INFO` may not be emitted.

For example:

```python
import logging

logging.debug("debug event")
logging.info("info event")
logging.warning("warning event")
```

The warning is visible under the default configuration, while the lower-severity events are not.

This is why configuring logging is an important part of an executable automation program.

---

## 6. Configure logging with `basicConfig()`

For simple applications, `logging.basicConfig()` provides a convenient way to configure the root logger.

```python
import logging

logging.basicConfig(level=logging.INFO)

logger = logging.getLogger(__name__)
logger.info("automation started")
```

A useful rule is to configure logging before emitting the application's log events.

For small scripts, `basicConfig()` may be enough. More complex applications can use the broader logging configuration facilities documented by Python.

---

## 7. Log to a file

`basicConfig()` can configure a file destination:

```python
import logging

logging.basicConfig(
    filename="automation.log",
    encoding="utf-8",
    level=logging.INFO,
)

logger = logging.getLogger(__name__)
logger.info("job started")
```

The Python HOWTO documents the `encoding` parameter for file logging.

A file destination can be useful for a local automation tool, but production environments may instead collect process output through their runtime, operating system, container platform, or logging infrastructure.

The repository does not assume a particular production logging platform.

---

## 8. Format log messages

You can control the displayed format with `format`:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(levelname)s:%(message)s",
)
```

For example, a message can include the logger name:

```python
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(name)s %(levelname)s %(message)s",
)
```

Commonly useful record information includes:

- timestamp
- logger name
- severity
- message

Use a format that supports the operational questions you need to answer.

---

## 9. Log variable data correctly

The logging API accepts the message format and its variable arguments separately:

```python
logger.info("processed %d files", file_count)
```

This is preferable to eagerly constructing the message with string interpolation for ordinary logging calls:

```python
logger.info("processed %d files", file_count)
```

The logging system performs the message formatting when the record is handled.

For structured operational information, keep the message clear and avoid putting secrets or unnecessarily large data into it.

---

## 10. Log exceptions with context

When handling an exception, `logger.exception()` can record the error with exception information.

```python
try:
    deploy()
except RuntimeError:
    logger.exception("deployment failed")
    raise
```

The logging documentation also supports `exc_info` when exception information needs to be added explicitly.

Logging an exception does not replace deciding what the application should do with the exception.

A useful distinction is:

```text
log the failure
      +
decide whether to recover, retry, propagate, or terminate
```

Do not automatically swallow an exception merely because it was logged.

---

## 11. Logger hierarchy

Logger names form a hierarchy separated by periods.

For example:

```text
myapp
├── myapp.deploy
└── myapp.config
```

A module can use:

```python
logger = logging.getLogger(__name__)
```

and its logger participates in that hierarchy.

Log records can propagate to handlers attached to ancestor loggers.

This lets an application configure logging centrally while individual modules simply obtain their own loggers.

---

## 12. Avoid duplicate log messages

Because log records can propagate through the logger hierarchy, attaching handlers at multiple levels can result in the same record being emitted more than once.

A common arrangement is:

```text
module loggers
      ↓
   propagate
      ↓
root/application handler
```

The official documentation notes that you generally do not need to attach handlers to more than one logger in a hierarchy when propagation is being used as intended.

When troubleshooting duplicate messages, inspect both handler placement and propagation settings.

---

## 13. Handlers and formatters

The four major logging components are worth remembering:

```text
Logger
  → creates/accepts logging events

Handler
  → sends records somewhere

Formatter
  → controls representation

Filter
  → applies additional selection rules
```

For example, a file handler and a console handler can be configured independently in a more advanced application.

The repository should introduce these components conceptually before relying on complicated logging configuration.

---

## 14. Logging and command-line tools

For a DevOps CLI, logging can provide operational diagnostics while ordinary output remains focused on user-facing results.

A simple pattern is:

```python
import logging

logger = logging.getLogger(__name__)


def main():
    logging.basicConfig(level=logging.INFO)
    logger.info("starting deployment")
    # perform work
    logger.info("deployment completed")
```

The logging level can also be selected from a CLI option.

The official HOWTO demonstrates converting a user-supplied level such as `INFO` or `DEBUG` into the corresponding logging value and recommends validating invalid input.

A repository design might expose:

```text
--log-level DEBUG
--log-level INFO
--log-level WARNING
```

but the exact CLI interface is a design decision for the application, not a Python requirement.

---

## 15. Logging in automation workflows

A useful automation sequence is:

```text
load configuration
      ↓
validate inputs
      ↓
log meaningful start event
      ↓
perform action
      ↓
validate result
      ↓
log outcome
```

For example:

```python
logger.info("starting configuration validation")

validate_config(config)

logger.info("configuration validation completed")
```

Avoid logging every line of a script. Logs should help an engineer understand what happened without creating unnecessary noise.

---

## 16. Logging subprocess operations

When combining `logging` with `subprocess`, log the operational event rather than blindly dumping every command and every output value.

For example:

```python
logger.info("running configuration check")

result = subprocess.run(
    ["tool", "check"],
    capture_output=True,
    text=True,
    check=False,
)

if result.returncode != 0:
    logger.error("configuration check failed: exit code %s", result.returncode)
```

If stdout or stderr is needed for diagnosis, decide deliberately what should be retained or exposed.

This is especially important when command output can contain credentials, tokens, connection strings, or other sensitive operational information.

---

## 17. Logging and exceptions are different

Logging records an event.

Exceptions represent a program-control mechanism for reporting failures.

For example:

```python
try:
    deploy()
except RuntimeError:
    logger.exception("deployment failed")
    raise
```

Here:

1. the exception is logged with context;
2. the exception is re-raised;
3. the caller still gets a failure signal.

This is different from:

```python
try:
    deploy()
except RuntimeError:
    logger.exception("deployment failed")
```

where the exception is not re-raised and may therefore be treated as handled.

Choose the behavior intentionally.

---

## 18. Avoid logging secrets

Do not blindly log:

- passwords
- access tokens
- private keys
- authentication headers
- secret environment variables
- complete secret-bearing configuration objects
- command arguments containing credentials

A log message such as:

```python
logger.info("using token %s", token)
```

can turn a secret into persistent operational data.

Instead, log the fact that an authenticated operation is occurring without exposing the credential itself:

```python
logger.info("starting authenticated API request")
```

Exactly what should be considered sensitive depends on the application and environment.

---

## 19. Avoid excessive logging

More logs do not automatically mean better observability.

Excessive logging can make important events harder to find and can create unnecessary storage and processing requirements.

Prefer events that answer useful questions:

- What operation started?
- What resource was targeted?
- What important decision was made?
- Did the operation succeed?
- If it failed, where and why?
- Was a retry performed?
- What final state was reached?

Keep high-volume diagnostic information at `DEBUG` when appropriate.

---

## 20. `print()` still has a place

Logging should not replace every `print()` call.

The Python Logging HOWTO distinguishes ordinary console output from application-event logging.

For a CLI, for example:

```python
print("Deployment ID: 12345")
```

may be intended as the program's user-facing result, while:

```python
logger.info("deployment request completed")
```

is an operational event.

Design the boundary deliberately so users and operators get the information they need.

---

## 21. Common mistakes

### Mistake 1: using `print()` for every operational event

Use logging when events need severity, configuration, or integration with application logging.

### Mistake 2: never configuring logging

Lower-severity events may not be emitted under the default threshold.

### Mistake 3: creating handlers everywhere

This can produce duplicate records when logger propagation is also enabled.

### Mistake 4: logging secrets

Logs often persist longer and travel farther than expected. Treat log content as operational data that requires care.

### Mistake 5: logging an exception and silently continuing

Recording an error does not decide whether the failure is recoverable.

### Mistake 6: using the wrong severity

Do not label every event `ERROR`. Use the level that matches the operational meaning.

### Mistake 7: logging huge command output by default

Capture and expose output intentionally, especially when external tools can produce sensitive or very large data.

### Mistake 8: assuming logger configuration is local

Logger hierarchy and propagation can cause configuration at higher levels to affect child loggers.

### Mistake 9: treating logs as a substitute for application state

Logs describe events. They should not be the only source of truth for important state that the application must reliably manage.

---

## 22. DevOps practice exercise

Build a small deployment-style automation script that:

1. creates a module-level logger with `logging.getLogger(__name__)`;
2. accepts a configurable log level;
3. validates the requested level;
4. logs the beginning of an operation at `INFO`;
5. logs detailed diagnostic information at `DEBUG`;
6. records an unexpected condition at `WARNING`;
7. records a failed operation at `ERROR`;
8. uses `logger.exception()` when handling an exception that should retain traceback information;
9. never logs credentials or secret environment variables;
10. keeps ordinary user-facing CLI output separate from operational logging.

Then run the script with different logging levels and observe which events are emitted.

---

## 23. Knowledge checks

Before moving on, you should be able to answer:

1. What problem does the `logging` module solve?
2. What is the difference between a logger, handler, formatter, and filter?
3. Why is `logging.getLogger(__name__)` commonly recommended?
4. What are the five standard logging levels?
5. What is the default root logging level?
6. What does `basicConfig()` provide?
7. Why should logging usually be configured before application events are emitted?
8. How do logger names form a hierarchy?
9. What is propagation?
10. Why can multiple handlers produce duplicate records?
11. When should you use `logger.exception()`?
12. Why is logging an exception different from handling the exception?
13. When is `print()` still appropriate?
14. Why should secrets never be blindly logged?
15. How should subprocess output be handled when it may contain sensitive information?
16. Why is `DEBUG` useful for detailed diagnostics?

---

## 24. Completion criteria

You are ready to move on when you can:

- create and use a module-level logger;
- choose an appropriate severity level;
- configure logging for a script;
- write logs to the console or a file;
- format log records;
- understand logger hierarchy and propagation;
- explain the roles of handlers and formatters;
- log exceptions without automatically swallowing them;
- separate user-facing output from operational logging;
- integrate logging with subprocess-based automation;
- avoid exposing secrets through logs.

---

## 25. Verified facts vs repository design

### Verified facts

The official Python documentation establishes the `logging` API, standard logging levels, logger hierarchy, handlers, formatters, filters, propagation behavior, `getLogger(__name__)`, `basicConfig()`, and exception-information support.

### Repository design decisions

The following are teaching choices for this repository:

- treating logging as an operational boundary in DevOps automation;
- separating ordinary CLI output from operational events;
- emphasizing secret-safe logging;
- recommending a start → action → result event pattern;
- using subprocess examples to connect logging with earlier automation topics.

These choices are not claims that Python or any particular production platform requires one universal logging architecture.

---

## 26. Authoritative references

- Python `logging` library reference: https://docs.python.org/3/library/logging.html
- Python Logging HOWTO: https://docs.python.org/3/howto/logging.html
- Python Logging Cookbook: https://docs.python.org/3/howto/logging-cookbook.html
