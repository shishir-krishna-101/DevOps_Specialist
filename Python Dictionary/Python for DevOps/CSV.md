# CSV

CSV (Comma-Separated Values) is a common interchange format for tabular data. Python's standard-library `csv` module provides readers, writers, dictionary-based readers and writers, and support for different CSV dialects.

> **Source-first note:** Python's official `csv` documentation is the source of truth for the APIs and behavior described here. The operational examples and exercises are repository demonstrations.

## 1. Mental model

Think of CSV processing as:

```text
CSV file → reader → rows → validation/transformation → writer → CSV file
```

A CSV file is text, but CSV parsing should not normally be implemented by simply calling `split(',')`. Fields can contain delimiters, quotes, and line breaks according to the dialect being used.

For DevOps work, CSV is useful for inventory exports, reports, migration inputs, batch-processing data, and simple operational handoffs.

## 2. The `csv` module

```python
import csv
```

The standard library provides:

- `csv.reader()` for sequence-oriented rows;
- `csv.writer()` for writing sequence-oriented rows;
- `csv.DictReader` for mapping rows to dictionaries;
- `csv.DictWriter` for writing dictionaries;
- dialects and formatting parameters for variations in CSV representation.

## 3. Reading rows with `csv.reader`

Open a CSV text file with `newline=''` when passing the file object to the CSV reader.

```python
import csv

with open("servers.csv", newline="") as csvfile:
    reader = csv.reader(csvfile)
    for row in reader:
        print(row)
```

Rows returned by `reader` are sequences of strings. The CSV reader does not generally infer application-specific data types for ordinary fields.

If the file contains:

```text
name,environment,status
api-01,prod,healthy
worker-01,stage,healthy
```

then each row can be processed as a sequence of fields.

## 4. Why `newline=''` matters

Python's official documentation recommends opening CSV files with `newline=''` when using `csv.reader()` or `csv.writer()` with a file object.

Use:

```python
with open("servers.csv", newline="") as csvfile:
    reader = csv.reader(csvfile)
```

and for output:

```python
with open("report.csv", "w", newline="") as csvfile:
    writer = csv.writer(csvfile)
```

Treat this as part of the standard file-handling pattern for this module.

## 5. Reading rows as dictionaries

`csv.DictReader` maps each row to a dictionary using field names.

```python
import csv

with open("servers.csv", newline="") as csvfile:
    reader = csv.DictReader(csvfile)
    for row in reader:
        print(row["name"], row["status"])
```

When `fieldnames` is omitted, the first row is used as the field names and is not returned as a normal data row.

Dictionary-based access is often easier to review than numeric indexes when the CSV has meaningful column names.

## 6. Explicit field names

You can provide field names yourself:

```python
import csv

fieldnames = ["name", "environment", "status"]

with open("servers.csv", newline="") as csvfile:
    reader = csv.DictReader(csvfile, fieldnames=fieldnames)
    for row in reader:
        print(row)
```

When field names are explicitly supplied, the first physical row is treated as data rather than automatically being consumed as a header.

## 7. Writing rows with `csv.writer`

Use `csv.writer()` when the data is naturally represented as sequences.

```python
import csv

rows = [
    ["name", "environment", "status"],
    ["api-01", "prod", "healthy"],
    ["worker-01", "stage", "healthy"],
]

with open("report.csv", "w", newline="") as csvfile:
    writer = csv.writer(csvfile)
    writer.writerows(rows)
```

For individual rows:

```python
writer.writerow(["api-02", "prod", "degraded"])
```

The writer converts non-string values using `str()` when writing. `None` is written as an empty string; this is not a reversible representation of `None`.

## 8. Writing dictionaries with `DictWriter`

Use `DictWriter` when your application data is already represented by dictionaries.

```python
import csv

fieldnames = ["name", "environment", "status"]

with open("report.csv", "w", newline="") as csvfile:
    writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerow({
        "name": "api-01",
        "environment": "prod",
        "status": "healthy",
    })
```

`fieldnames` determines the output column order.

By default, `DictWriter` raises `ValueError` when a dictionary contains a key that is not present in `fieldnames`. This can be useful because unexpected columns are detected rather than silently discarded.

If intentionally ignoring extra keys is appropriate, `extrasaction='ignore'` can be specified.

## 9. CSV dialects

CSV is not a single perfectly uniform representation. Applications can vary in delimiter, quoting, line terminators, and related formatting details.

The `csv` module models these differences with **dialects** and formatting parameters.

For example, a tab-delimited file can be handled explicitly:

```python
import csv

with open("servers.tsv", newline="") as csvfile:
    reader = csv.reader(csvfile, delimiter="\t")
    for row in reader:
        print(row)
```

Use explicit formatting parameters when the producer's format is known.

## 10. Delimiters and quoting

Do not assume the delimiter is always a comma.

```python
reader = csv.reader(csvfile, delimiter=";")
```

Quoting is also part of CSV representation. The module supports quoting constants such as:

```python
csv.QUOTE_MINIMAL
csv.QUOTE_ALL
csv.QUOTE_NONNUMERIC
csv.QUOTE_NONE
```

Use the documented quoting mode appropriate to the producer or consumer rather than manually removing quote characters.

## 11. Custom dialects

If a particular CSV representation is reused, a dialect can be registered and named.

```python
import csv

csv.register_dialect(
    "operations",
    delimiter=";",
    quotechar='"',
)

with open("inventory.csv", newline="") as csvfile:
    reader = csv.reader(csvfile, dialect="operations")
    for row in reader:
        print(row)
```

Repository design decision: prefer explicit formatting parameters for one-off integrations and a named dialect when the same format is used repeatedly across the repository's automation.

## 12. `Sniffer`

Python provides `csv.Sniffer` for attempting to deduce a CSV dialect from a sample.

```python
import csv

with open("input.csv", newline="") as csvfile:
    sample = csvfile.read(1024)
    dialect = csv.Sniffer().sniff(sample)
    csvfile.seek(0)
    reader = csv.reader(csvfile, dialect)
    for row in reader:
        print(row)
```

`Sniffer` is heuristic. The official documentation specifically notes that header detection can produce false positives and false negatives.

For production automation, prefer a known and documented input format when one exists instead of depending on automatic detection.

## 13. Header handling

If the input contract says the first row is a header, `DictReader` is often convenient:

```python
reader = csv.DictReader(csvfile)
```

For a sequence-oriented reader, handle the header explicitly:

```python
reader = csv.reader(csvfile)
header = next(reader)
for row in reader:
    ...
```

Validate required headers before processing operational data when the input contract depends on them.

## 14. Validation before processing

CSV parsing and data validation are separate concerns.

The CSV parser can tell you where fields and rows are. Your automation still needs to decide whether those values satisfy the application's requirements.

Example:

```python
required = {"name", "environment", "status"}

with open("servers.csv", newline="") as csvfile:
    reader = csv.DictReader(csvfile)
    headers = set(reader.fieldnames or [])

    missing = required - headers
    if missing:
        raise ValueError(f"Missing required columns: {sorted(missing)}")
```

Repository design decision: fail early when required columns are absent instead of allowing malformed operational input to flow deeper into the automation.

## 15. Values are text unless you convert them

CSV fields are normally read as strings.

If a column represents a number, perform the conversion deliberately:

```python
count = int(row["count"])
```

If a column represents a boolean or an enum-like value, define the accepted representations explicitly rather than relying on Python truthiness.

Example:

```python
status = row["status"].strip()
if status not in {"healthy", "degraded", "failed"}:
    raise ValueError(f"Unsupported status: {status}")
```

## 16. CSV and DevOps automation

CSV commonly appears at boundaries between systems rather than as a core configuration language.

Useful patterns include:

- importing a server or service inventory;
- producing deployment or audit reports;
- processing a batch of operational requests;
- transforming an exported table into API input;
- generating a summary for humans to inspect.

Example inventory processing:

```python
import csv

with open("inventory.csv", newline="") as csvfile:
    reader = csv.DictReader(csvfile)

    for row in reader:
        name = row["name"].strip()
        environment = row["environment"].strip()

        if not name or not environment:
            continue

        print(f"{name}: {environment}")
```

The exact validation and action should be determined by the application's input contract.

## 17. CSV versus structured configuration

CSV is well suited to tabular records. It is not automatically the best representation for nested configuration.

Repository design decision:

- Use CSV for rows and columns.
- Use JSON or another appropriate structured format when nested structure is important.
- Do not use regex or manual string splitting as a substitute for CSV parsing.

## 18. Large files and streaming

`csv.reader()` and `csv.DictReader` can be consumed row by row, which fits operational processing of files that should not be loaded entirely into memory.

```python
import csv

with open("large-inventory.csv", newline="") as csvfile:
    reader = csv.DictReader(csvfile)
    for row in reader:
        process(row)
```

Repository design decision: prefer row-by-row processing for large operational inputs when the workflow does not require the entire dataset in memory.

## 19. Common mistakes

### Mistake 1: Using `split(',')`

Manual splitting does not correctly model CSV quoting and dialect behavior.

Use `csv.reader()` or `csv.DictReader()` instead.

### Mistake 2: Omitting `newline=''`

Follow the official CSV file-opening guidance when using file objects with the reader or writer.

### Mistake 3: Assuming every CSV uses commas

Confirm the delimiter and other dialect properties from the input contract.

### Mistake 4: Assuming values have already been typed

CSV fields are generally strings. Convert and validate them explicitly.

### Mistake 5: Trusting `Sniffer` as a contract

Automatic dialect detection is heuristic. Known formats should be configured explicitly where possible.

### Mistake 6: Silently ignoring unexpected columns

For operational data, unexpected fields may indicate an upstream schema change. `DictWriter` defaults to raising for extra dictionary keys; choose a different behavior only when the integration contract calls for it.

### Mistake 7: Mixing parsing and side effects

Separate CSV parsing and validation from destructive or externally visible operations where practical. This makes malformed input easier to detect before changes are made.

## 20. Practical exercise: inventory validator

Create a script that reads:

```text
name,environment,status
api-01,prod,healthy
worker-01,stage,healthy
```

Requirements:

1. Read with `DictReader`.
2. Require `name`, `environment`, and `status` columns.
3. Reject blank names.
4. Allow only `dev`, `stage`, or `prod` environments.
5. Allow only `healthy`, `degraded`, or `failed` statuses.
6. Report invalid rows without performing external side effects.

## 21. Practical exercise: CSV report generator

Build a small report generator that receives a sequence of deployment records and writes:

```text
service,environment,status
```

Requirements:

- use `DictWriter`;
- write the header;
- preserve the declared field order;
- verify that unexpected keys are handled intentionally;
- write the file using the standard CSV newline pattern.

## 22. Practical exercise: format boundary

Create two readers for the same logical inventory:

1. a comma-delimited format;
2. a semicolon-delimited format.

Keep the validation and processing logic shared while isolating only the CSV dialect configuration.

This demonstrates why parsing configuration should be separated from application logic.

## 23. Knowledge check

Before moving on, you should be able to explain:

1. Why `csv.reader()` is preferable to `split(',')` for CSV data.
2. Why `newline=''` is used when opening CSV files for the module.
3. The difference between `reader` and `DictReader`.
4. The difference between `writer` and `DictWriter`.
5. What a CSV dialect represents.
6. Why `Sniffer` should not automatically be treated as authoritative input validation.
7. Why CSV values generally require explicit application-level type conversion.
8. When CSV is a better fit than JSON or another structured format.

## 24. Completion criteria

You have completed this topic when you can:

- read CSV rows with `reader`;
- read named columns with `DictReader`;
- write sequence rows with `writer`;
- generate named-column output with `DictWriter`;
- configure delimiters and quoting intentionally;
- understand dialects;
- validate headers and field values;
- process large files row by row;
- recognize malformed or unexpected input before performing side effects.

## 25. Verified facts vs repository design

### Verified facts

- Python's standard-library `csv` module reads and writes tabular CSV-style data.
- `reader()` and `writer()` operate on sequence-oriented rows.
- `DictReader` and `DictWriter` provide dictionary-based row handling.
- CSV files opened as file objects for the reader or writer should use `newline=''`.
- CSV dialects describe variations in delimiters, quoting, and related formatting behavior.
- `Sniffer` performs heuristic format detection and header detection.

### Repository design decisions

- Operational examples use CSV primarily at data-import and reporting boundaries.
- Required columns and allowed values should be validated before side effects.
- Known input formats should be configured explicitly when practical.
- Row-by-row processing is preferred for large files when full in-memory loading is unnecessary.

### Demonstrations

All inventories, statuses, field names, and example records in this file are teaching demonstrations created for this repository. They do not describe any vendor's production schema.

## 26. Authoritative references

- Python `csv` — CSV File Reading and Writing: https://docs.python.org/3/library/csv.html
- Python Standard Library: https://docs.python.org/3/library/
