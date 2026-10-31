# Regular Expressions

Regular expressions (regex) are patterns used to search, match, extract, split, and replace text. In DevOps work, they are useful when structured parsing is unavailable or when a task needs a compact text pattern—for example, extracting fields from logs, locating identifiers, or validating a constrained text format.

> **Source-first note:** The Python `re` module is the authoritative source for the Python regex API and syntax used here. Examples in this file are repository demonstrations, not official product behavior.

## 1. Mental model

A regular expression has two parts:

- **Pattern** — describes text you want to recognize.
- **Input text** — the string against which the pattern is applied.

The `re` module provides functions and compiled pattern objects for performing these operations.

Think of regex as a small language for describing text structure:

```text
input text → regex engine → match object / matches / replacement
```

For DevOps automation, decide first whether the input is really text that should be parsed with regex. If the data has a reliable structured format such as JSON, prefer its parser instead of using regex to reconstruct structure.

## 2. Raw strings

Python string escaping and regex escaping are separate concerns. Raw string literals are commonly used for regex patterns so that backslashes are passed to the regex engine without Python interpreting most backslash escapes first.

```python
pattern = r"\bERROR\b"
```

Compare this with ordinary Python strings carefully when a pattern contains backslashes. The `r"..."` form is usually easier to read for regex-heavy code.

## 3. Literals and metacharacters

Some characters match themselves:

```python
r"ERROR"
r"timeout"
r"server-01"
```

Other characters have special regex meaning. Common metacharacters include:

```text
.  ^  $  *  +  ?  {  }  [  ]  \  |  (  )
```

If a metacharacter must be treated literally, escape it or use `re.escape()` when constructing a pattern from text that should be matched literally.

Example:

```python
import re

text = "node-01.example.com"
pattern = re.escape("node-01.example.com")

match = re.fullmatch(pattern, text)
print(bool(match))
```

## 4. Character classes

Character classes describe one character from a set or range.

```python
r"[abc]"       # a, b, or c
r"[0-9]"       # one ASCII digit from 0 through 9
r"[A-F]"       # one uppercase letter from A through F
r"[^#]"        # one character other than #
```

Use character classes when the allowed character set is the important part of the pattern.

## 5. Quantifiers

Quantifiers control how many times a preceding expression may occur.

Common forms include:

```text
*       zero or more
+       one or more
?       zero or one
{m}     exactly m
{m,n}   from m through n
```

For example:

```python
r"a+"
r"\d{4}"
r"[0-9]{1,3}"
```

Be deliberate about quantifiers. A broad expression can match much more text than intended.

## 6. Anchors

Anchors describe positions rather than consuming ordinary characters.

```text
^   start of the string, or line with MULTILINE
$   end of the string, or line with MULTILINE
\A  start of the string
\Z  end of the string
```

For complete validation of an input value, `re.fullmatch()` is often clearer than manually adding anchors.

Example:

```python
import re

if re.fullmatch(r"[A-Z]{3}-[0-9]{4}", "OPS-2026"):
    print("valid")
```

## 7. Groups and captures

Parentheses create groups. Capturing groups let you retrieve the part of the input matched by that group.

```python
import re

text = "service=payments status=failed"
match = re.search(r"service=(\w+) status=(\w+)", text)

if match:
    print(match.group(1))
    print(match.group(2))
```

Use non-capturing groups when grouping is needed but the matched text does not need to be returned:

```python
r"(?:dev|stage|prod)-\d+"
```

## 8. Named groups

Named groups make extraction code easier to read when a pattern has multiple fields.

```python
import re

text = "service=payments status=failed"
pattern = r"service=(?P<service>\w+) status=(?P<status>\w+)"

match = re.search(pattern, text)

if match:
    values = match.groupdict()
    print(values["service"])
    print(values["status"])
```

For operational scripts, named groups are usually preferable when the extracted fields have meaningful names.

## 9. Core matching operations

The `re` module exposes several common operations.

### `re.search()`

Find the first location where the pattern produces a match.

```python
match = re.search(r"ERROR", log_line)
```

### `re.match()`

Try to match at the beginning of the string.

```python
match = re.match(r"ERROR", log_line)
```

### `re.fullmatch()`

Require the entire string to match the pattern.

```python
match = re.fullmatch(r"[a-z0-9-]+", hostname)
```

### `re.findall()`

Return all non-overlapping matches in the input.

```python
ids = re.findall(r"ticket-[0-9]+", text)
```

### `re.finditer()`

Iterate over match objects for all non-overlapping matches.

```python
for match in re.finditer(r"ticket-[0-9]+", text):
    print(match.group(), match.start(), match.end())
```

### `re.split()`

Split text using a regex pattern.

```python
parts = re.split(r"\s*,\s*", "dev, stage, prod")
```

### `re.sub()` and `re.subn()`

Replace matches. `subn()` also reports the number of substitutions made.

```python
cleaned = re.sub(r"\s+", " ", text).strip()
```

Use substitutions carefully when modifying configuration, logs, or generated files. A pattern that matches too broadly can change data that should have remained untouched.

## 10. Compiled patterns

Use `re.compile()` when a pattern will be reused.

```python
import re

ERROR_PATTERN = re.compile(r"\bERROR\b")

for line in lines:
    if ERROR_PATTERN.search(line):
        print(line)
```

Compiled pattern objects provide methods corresponding to common matching operations and make reusable patterns easier to name and organize.

## 11. Flags

Flags change how a pattern is interpreted or applied. Common flags include:

```python
re.IGNORECASE
re.MULTILINE
re.DOTALL
```

Example:

```python
pattern = re.compile(r"^error:.*$", re.IGNORECASE | re.MULTILINE)
```

Use flags intentionally and document them when they materially affect parsing behavior.

## 12. Match objects

When a match succeeds, Python returns a match object. Useful information includes:

```python
match.group()
match.start()
match.end()
match.span()
```

For named groups:

```python
match.groupdict()
```

Always handle the no-match case before calling methods on a possibly missing match object.

## 13. Regex in DevOps automation

### Log extraction

A regex can extract a small, stable field from a log line:

```python
import re

line = "2026-10-31T10:15:00Z service=api status=500"
pattern = re.compile(
    r"service=(?P<service>\w+) status=(?P<status>[0-9]+)"
)

match = pattern.search(line)
if match:
    print(match.groupdict())
```

### Identifier detection

```python
matches = re.findall(r"INC-[0-9]{6}", text)
```

### Whitespace normalization

```python
normalized = re.sub(r"\s+", " ", text).strip()
```

### Constrained input validation

```python
valid = re.fullmatch(r"[A-Z]{2}-[0-9]{4}", value) is not None
```

The pattern should express the actual requirement. Do not treat a regex match as proof that a value is semantically valid beyond what the pattern checks.

## 14. Validation vs extraction

These are different jobs.

**Validation:** Does the entire input conform to a specified format?

```python
re.fullmatch(pattern, value)
```

**Extraction:** Where are the fields or tokens inside a larger input?

```python
re.search(pattern, text)
re.findall(pattern, text)
re.finditer(pattern, text)
```

Keeping the intent explicit makes automation easier to review and test.

## 15. Regex and structured data

Regex is a text-pattern tool, not a general-purpose parser for every data format.

Prefer a format-specific parser when the input is structured and a reliable parser exists. For example, this repository already uses Python's JSON support in the **JSON** topic rather than teaching regex as a replacement for JSON parsing.

Repository design decision:

- Use regex for targeted text matching, extraction, normalization, and constrained validation.
- Use dedicated parsers for structured formats where practical.

## 16. Common mistakes

### Mistake 1: Forgetting raw strings

Backslashes can be interpreted by Python string syntax before the regex engine sees them.

Prefer:

```python
pattern = r"\d+"
```

when the raw-string form improves clarity.

### Mistake 2: Using `match()` when you need a search

`re.match()` works at the beginning of the string. If the target may occur later, use `re.search()` or another operation that matches the requirement.

### Mistake 3: Using `search()` for whole-input validation

If the complete value must conform to a pattern, use `re.fullmatch()` rather than accepting a substring match.

### Mistake 4: Making the pattern broader than necessary

Patterns such as `.*` can consume large portions of input and may make extraction ambiguous. Prefer explicit structure when the format is known.

### Mistake 5: Ignoring input boundaries

When extracting from operational data, define what counts as one record and test cases containing unexpected whitespace, missing fields, or additional text.

### Mistake 6: Treating regex as a universal parser

If the input is structured data, use the appropriate parser instead of constructing increasingly complex regex patterns.

### Mistake 7: Assuming a successful match means semantic validity

Regex checks only what its pattern expresses. Additional application-level checks may still be required.

## 17. Practical exercise: log field extractor

Write a function that receives lines like:

```text
2026-10-31T10:15:00Z service=api status=500
2026-10-31T10:16:00Z service=worker status=200
```

Requirements:

1. Extract `service` and `status`.
2. Return the values using named groups.
3. Ignore lines that do not match the expected structure.
4. Write tests for a valid line, a missing field, and an unexpected status format.

## 18. Practical exercise: identifier scanner

Given a text file containing incident references such as `INC-123456`, use `finditer()` to report each identifier and its character position.

Then extend the exercise to:

- count occurrences;
- deduplicate identifiers;
- preserve the first position where each identifier appeared.

## 19. Practical exercise: safe normalization

Create a function that normalizes repeated whitespace in a text value without changing non-whitespace characters.

Test it with:

- multiple spaces;
- tabs;
- newlines;
- already-normalized text;
- an empty string.

## 20. Knowledge check

Before moving on, you should be able to explain:

1. Why raw strings are commonly used for regex patterns.
2. The difference between `search()`, `match()`, and `fullmatch()`.
3. When to use `findall()` versus `finditer()`.
4. What capturing and named groups provide.
5. Why `re.compile()` is useful for reusable patterns.
6. How `re.sub()` differs from `re.subn()`.
7. When a dedicated parser is preferable to regex.
8. Why a regex match does not automatically establish semantic validity.

## 21. Completion criteria

You have completed this topic when you can:

- read and write basic regex patterns;
- use character classes and quantifiers;
- use anchors appropriately;
- extract data with groups and named groups;
- choose the appropriate `re` matching operation;
- use compiled patterns and flags;
- perform controlled substitutions;
- test regex-based automation against malformed and unexpected input;
- recognize when regex should be replaced by a structured-data parser.

## 22. Verified facts vs repository design

### Verified facts

- Python's standard-library `re` module provides regular-expression operations.
- The module includes search, match, full-match, splitting, substitution, compilation, and matching APIs.
- Python supports capturing groups and named groups in regular expressions.
- Regex flags can modify matching behavior.

### Repository design decisions

- DevOps examples focus on logs, identifiers, text normalization, and constrained validation.
- Structured data should normally be handled by its dedicated parser rather than by regex.
- Exercises emphasize safe handling of malformed and unexpected operational input.

### Demonstrations

The code examples and exercises in this file are teaching demonstrations created for this repository. They are not claims about any particular vendor, platform, log format, or production environment.

## 23. Authoritative references

- Python `re` — Regular expression operations: https://docs.python.org/3/library/re.html
- Python Regular Expression HOWTO: https://docs.python.org/3/howto/regex.html
- Python Standard Library: https://docs.python.org/3/library/
