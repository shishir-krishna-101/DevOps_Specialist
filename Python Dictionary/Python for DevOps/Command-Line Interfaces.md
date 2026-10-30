# Command-Line Interfaces

A command-line interface (CLI) is a boundary between a user or automation system and a program. For DevOps work, a good CLI should make inputs explicit, validate them early, provide useful help, and return a meaningful process result.

> **Source note:** This topic is based on the official Python documentation for `argparse`, `sys.argv`, and Python command-line invocation. The examples are teaching demonstrations created for this repository.

---

## 1. The mental model

Treat a CLI as an input boundary:

```text
command line
     ↓
argument parsing
     ↓
validation
     ↓
application logic
     ↓
result / output
     ↓
process exit status
```

Keep command-line parsing separate from the work performed by the program.

For example:

```python
def parse_args():
    ...


def run(config):
    ...


def main():
    args = parse_args()
    run(args)
```

This separation makes the core logic easier to test without constructing a real command line.

---

## 2. `sys.argv`

Python exposes the command-line arguments passed to a script through `sys.argv`.

```python
import sys

print(sys.argv)
```

`sys.argv` is a list. The first element, `sys.argv[0]`, represents the program or script entry used for the invocation; the exact value depends on how Python was invoked.

For example, if a script is invoked with arguments after the script name, those arguments are available in the remaining elements.

The official documentation describes `sys.argv` as the list of command-line arguments passed to a Python script.

---

## 3. Why use `argparse`?

Python's standard library provides `argparse` for implementing command-line interfaces.

It can:

- define positional arguments
- define options such as `--environment`
- define boolean flags
- convert argument values with supported type functions
- provide defaults
- restrict values with choices
- generate help and usage messages
- report invalid arguments
- organize related commands with subparsers

The parser works with the arguments represented by `sys.argv`.

```python
import argparse

parser = argparse.ArgumentParser(description="Inspect a service")
parser.add_argument("service")

args = parser.parse_args()
print(args.service)
```

The resulting object is an `argparse.Namespace` containing the parsed values.

---

## 4. Positional arguments

A positional argument is identified by its position in the command.

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("service")

args = parser.parse_args()
print(args.service)
```

Conceptually:

```text
program service-name
        ^^^^^^^^^^^^
        positional value
```

Use positional arguments when the value is naturally part of the command's required target.

Whether a value should be positional or optional is a CLI design decision.

---

## 5. Optional arguments

Options normally use a flag name such as `--environment`.

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--environment")

args = parser.parse_args()
print(args.environment)
```

A user can provide a value explicitly:

```text
program --environment production
```

Short and long option forms can also be defined together:

```python
parser.add_argument("-e", "--environment")
```

Keep option names descriptive. A CLI is an interface that other people and automation may depend on.

---

## 6. Boolean flags

A common CLI requirement is an on/off option.

`argparse` supports this with actions such as `store_true`.

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--verbose", action="store_true")

args = parser.parse_args()

if args.verbose:
    print("verbose mode enabled")
```

When the flag is present, the parsed value is true; when it is absent, the corresponding default is false for this action.

Use boolean flags for clear switches rather than requiring users to type values such as `--verbose true` unless the interface specifically needs that representation.

---

## 7. Defaults

Optional arguments can have defaults.

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--environment", default="development")

args = parser.parse_args()
print(args.environment)
```

Defaults are part of the CLI's behavior and should be chosen deliberately.

For infrastructure automation, avoid defaults that could silently target an unintended environment.

For example, a deployment tool should make its target environment sufficiently explicit for the risk involved.

That safety recommendation is a repository design principle, not a rule imposed by `argparse`.

---

## 8. Converting argument values with `type`

Command-line arguments arrive from the command line as text. `argparse` can apply a conversion function through `type`.

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--retries", type=int, default=3)

args = parser.parse_args()
print(args.retries)
```

The resulting `args.retries` value is an integer when parsing succeeds.

Use conversion at the input boundary when the application expects a specific basic type.

For more complex validation, parse first and then apply explicit application-level checks.

---

## 9. Restricting values with `choices`

When an argument must be one of a known set of values, `choices` can express that constraint.

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument(
    "--environment",
    choices=["development", "staging", "production"],
    required=True,
)

args = parser.parse_args()
print(args.environment)
```

This moves a simple allowed-value check into the CLI parser.

The set of allowed values is application-specific. Do not copy these environment names into another tool and assume its accepted values are the same.

---

## 10. Required options

An optional-style argument can be required explicitly.

```python
parser.add_argument("--environment", required=True)
```

This is different from a positional argument: the user still supplies the option by its flag name.

For operational commands, requiring a high-impact value can be preferable to silently choosing a dangerous default.

The appropriate choice depends on the command's purpose and risk.

---

## 11. Help and usage messages

`ArgumentParser` automatically provides help behavior.

```python
parser = argparse.ArgumentParser(
    description="Inspect service configuration"
)
```

A CLI should make its interface discoverable:

```text
program --help
```

`argparse` generates usage and help information from the parser definition.

Add useful descriptions to arguments rather than relying entirely on option names.

---

## 12. A small complete CLI

A simple DevOps-oriented example:

```python
import argparse


def parse_args():
    parser = argparse.ArgumentParser(
        description="Validate a service configuration"
    )
    parser.add_argument("config")
    parser.add_argument(
        "--environment",
        choices=["development", "staging", "production"],
        required=True,
    )
    parser.add_argument("--verbose", action="store_true")
    return parser.parse_args()


def main():
    args = parse_args()

    if args.verbose:
        print(f"config={args.config}")
        print(f"environment={args.environment}")

    # Application logic would use args.config and args.environment here.


if __name__ == "__main__":
    main()
```

The structure is intentionally simple:

```text
parse_args()
     ↓
validated Namespace
     ↓
main logic
```

---

## 13. Subcommands

Larger CLIs often contain related commands.

For example:

```text
program deploy ...
program status ...
program rollback ...
```

`argparse` supports this style through subparsers.

A simplified structure is:

```python
import argparse

parser = argparse.ArgumentParser()
subparsers = parser.add_subparsers(dest="command")

subparsers.add_parser("status")
subparsers.add_parser("deploy")

args = parser.parse_args()

if args.command == "status":
    print("status command")
elif args.command == "deploy":
    print("deploy command")
```

Subcommands are useful when a tool has multiple distinct operations with different arguments.

Keep each command's behavior separate rather than building one function containing every operation.

---

## 14. Keep parsing separate from side effects

Avoid putting external operations directly into argument definitions or parsing code.

Prefer:

```text
CLI arguments
     ↓
parse
     ↓
validate
     ↓
configuration object
     ↓
perform action
```

over:

```text
parse one argument
     ↓
call cloud API
     ↓
parse another argument
     ↓
modify infrastructure
```

The first structure is easier to test and reason about.

This is a repository design decision.

---

## 15. CLI input is untrusted input

A CLI can be invoked by a human, CI system, scheduler, or another program.

Do not assume that an argument is valid merely because the parser accepted its syntax.

For example:

```python
parser.add_argument("--port", type=int)
```

confirms that the value can be converted to an integer, but an application may still need to check whether the resulting integer is acceptable for its own operation.

A useful pattern is:

```text
parse type
   ↓
validate range / semantics
   ↓
perform action
```

---

## 16. File arguments

A CLI can accept paths as ordinary arguments.

```python
parser.add_argument("config")
```

The parser does not automatically prove that the referenced file exists or is readable.

Use the appropriate filesystem APIs after parsing:

```python
from pathlib import Path

path = Path(args.config)

if not path.is_file():
    raise ValueError(f"configuration file not found: {path}")
```

For more detailed file handling, see the repository topics on `pathlib` and `Files`.

---

## 17. Environment variables and CLI arguments

CLI arguments and environment variables are separate configuration sources.

A program may intentionally support both:

```text
defaults
   ↓
configuration file
   ↓
environment
   ↓
CLI arguments
```

But this precedence is not automatic or universal.

If an application supports multiple sources, define and document its precedence explicitly.

For example, the CLI might override an environment-derived default, but that is an application design decision.

---

## 18. Avoid secrets in command lines

Command-line arguments can be inappropriate for sensitive values.

Avoid designing commands such as:

```text
program --password super-secret-value
```

when the surrounding platform provides a better secret-handling mechanism.

Prefer a design where sensitive material is obtained through an appropriate protected configuration or secret-management boundary.

Do not print the complete parsed `Namespace` when it may contain credentials or tokens.

This is security-oriented repository guidance; the exact secret-management mechanism depends on the platform.

---

## 19. Error handling

`argparse` reports invalid command-line arguments and provides usage information.

For example, a value that cannot satisfy an argument's configured conversion or choice constraint is rejected by the parser.

Application-level validation may still be needed after parsing.

Keep the distinction clear:

```text
argument parsing
      ↓
syntax / basic input constraints
      ↓
application validation
      ↓
operation
```

Do not catch parser failures broadly and continue as if the CLI were valid.

---

## 20. Testing a CLI

Design parsing as a separate function so it can be tested independently.

For example:

```python
def parse_args(argv=None):
    parser = argparse.ArgumentParser()
    parser.add_argument("--retries", type=int, default=3)
    return parser.parse_args(argv)
```

The optional `argv` parameter lets tests supply controlled argument values rather than relying on the process's real command line.

Example test idea:

```python
args = parse_args(["--retries", "5"])
assert args.retries == 5
```

The application entry point can still call:

```python
args = parse_args()
```

when it should parse the actual command line.

---

## 21. Exit status and automation

A CLI is often consumed by automation, so the process result matters in addition to printed output.

Conceptually:

```text
success → successful process result
failure → non-success process result
```

Python programs can communicate process termination through mechanisms such as `SystemExit` and `sys.exit()`.

The exact exit-code contract should be defined by the application. Do not assume that a particular numeric value has universal meaning beyond the conventions documented by the operating environment and the program.

For DevOps automation, make failure states observable to the calling system rather than merely printing an error and continuing.

---

## 22. CLI design for DevOps tools

A practical DevOps CLI should answer these questions clearly:

```text
What operation is being performed?
What target is affected?
What configuration is being used?
What values are required?
What values have safe defaults?
How can the user see help?
How is invalid input reported?
What happens on failure?
What process result indicates success or failure?
```

For high-impact operations, make important targets explicit.

For example, a deployment command should make the target environment and application sufficiently clear before performing the operation.

These are repository design principles rather than `argparse` requirements.

---

## 23. Common mistakes

### Mistake 1: manually parsing `sys.argv` for a growing CLI

Manual parsing can become difficult to maintain as options and validation rules grow.

Use `argparse` when its behavior fits the application's requirements.

### Mistake 2: treating parsing as complete validation

A successfully parsed argument can still violate application-level rules.

### Mistake 3: unsafe defaults

A default that silently targets production or another high-impact resource can be dangerous.

### Mistake 4: mixing parsing and side effects

Keep argument parsing separate from cloud, infrastructure, filesystem, or network operations.

### Mistake 5: logging sensitive arguments

Do not print secrets merely because they were supplied through the CLI.

### Mistake 6: making every argument optional

A command should communicate which information is genuinely required.

### Mistake 7: testing only through a shell

Test the parser and application logic separately so invalid inputs can be exercised deterministically.

### Mistake 8: inventing tool-specific behavior

A generic Python CLI should not claim that its arguments behave like Kubernetes, Terraform, AWS, or another tool unless that behavior has been verified against that tool's documentation.

---

## 24. DevOps practice exercise

Build a CLI called `config-check` with this conceptual interface:

```text
config-check CONFIG --environment ENV [--verbose]
```

Requirements:

1. Accept a required configuration path.
2. Require an environment value.
3. Restrict the environment to the values documented by your exercise.
4. Provide `--verbose` as an optional flag.
5. Display useful help.
6. Parse arguments separately from configuration loading.
7. Validate the file after parsing the CLI.
8. Do not print sensitive configuration values.
9. Return a meaningful process result when validation fails.
10. Test the parser with a supplied argument list.

---

## 25. Knowledge checks

Before moving on, you should be able to answer:

1. What is stored in `sys.argv`?
2. Why is `argparse` useful compared with manually indexing `sys.argv`?
3. What is a positional argument?
4. What is an optional argument?
5. How do boolean flags work with `argparse`?
6. What does the `type` argument do?
7. When is `choices` useful?
8. Why should CLI parsing be separated from application logic?
9. Why is a parsed argument not necessarily fully validated application input?
10. Why should sensitive values generally not be placed directly on a command line?
11. How can a parser be tested without using the real process command line?
12. Why does process exit status matter to DevOps automation?

---

## 26. Completion criteria

You are ready to move on when you can:

- explain `sys.argv`
- create a basic `argparse` parser
- define positional and optional arguments
- create boolean flags
- define defaults deliberately
- convert simple argument types
- constrain values with `choices`
- create help and usage information
- organize related operations with subcommands
- separate parsing from application actions
- test parsing with controlled argument lists
- design a CLI that behaves predictably in automation
- treat secrets and high-impact targets carefully

---

## 27. Source classification

### Verified facts

The behavior of `sys.argv`, Python command-line invocation, and the `argparse` interfaces described here is based on the official Python documentation.

### Repository design decisions

The separation of parsing, validation, configuration, and side effects; explicit high-impact targets; and cautious handling of secrets are teaching and repository design principles.

### Demonstrations

The CLI examples are generic demonstrations. They are not claims about the command-line interfaces of external DevOps tools.

---

## 28. Authoritative references

- Python `argparse`: https://docs.python.org/3/library/argparse.html
- Python `sys.argv`: https://docs.python.org/3/library/sys.html#sys.argv
- Python command-line usage: https://docs.python.org/3/using/cmdline.html
- Python `sys.exit()`: https://docs.python.org/3/library/sys.html#sys.exit
