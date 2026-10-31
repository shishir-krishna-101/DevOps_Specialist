# Dates and Times

Python's standard library provides `datetime` for manipulating dates, times, timestamps, durations, and time-zone-aware values. Date/time handling matters in DevOps because automation frequently works with schedules, log timestamps, retention windows, deployment records, certificates, and time-based comparisons.

> **Source-first note:** The examples below are repository demonstrations. Python behavior and API details are verified against the official `datetime` and `zoneinfo` documentation.

## 1. Mental model

Separate these concepts:

- **Date** — calendar date such as `2026-10-31`.
- **Time** — time of day.
- **Datetime** — date plus time.
- **Duration** — an amount of time represented by `timedelta`.
- **Time zone** — rules that relate local times to UTC and other zones.

A useful DevOps model is:

```text
calendar value → datetime object → comparison/arithmetic → formatted output
                         ↓
                    time zone
```

Do not treat a formatted timestamp string as equivalent to a datetime object. Parse it into the appropriate type before doing date/time arithmetic.

## 2. Importing `datetime`

The official documentation notes that the module and the `datetime` class have the same name. A clear repository convention is:

```python
import datetime as dt
```

Then the types are explicit:

```python
today = dt.date.today()
now = dt.datetime.now()
delta = dt.timedelta(days=7)
```

## 3. Dates

A `date` represents a calendar date and exposes `year`, `month`, and `day`.

```python
import datetime as dt

release_date = dt.date(2026, 10, 31)
print(release_date.year)
print(release_date.month)
print(release_date.day)
```

Get today's local date with:

```python
today = dt.date.today()
```

Dates can be compared:

```python
if release_date > today:
    print("release is in the future")
```

## 4. Times

A `time` represents a time of day independently of a particular date.

```python
import datetime as dt

start = dt.time(9, 30)
print(start.hour)
print(start.minute)
```

For operational automation, remember that a time without a date does not identify a unique moment in a deployment timeline.

## 5. Datetimes

A `datetime` combines date and time information.

```python
import datetime as dt

scheduled = dt.datetime(2026, 10, 31, 18, 30)
print(scheduled.year)
print(scheduled.hour)
```

The current local datetime can be obtained with:

```python
now = dt.datetime.now()
```

For UTC-aware work, use an aware datetime rather than assuming that a naive value represents UTC.

## 6. Naive vs aware datetimes

Python distinguishes **naive** and **aware** date/time objects.

- A naive `datetime` does not contain enough time-zone information to identify its position relative to UTC unambiguously.
- An aware `datetime` has enough time-zone information to locate itself relative to other aware datetimes.

Example of an explicitly UTC-aware value:

```python
import datetime as dt

now_utc = dt.datetime.now(dt.timezone.utc)
print(now_utc)
```

Do not silently label a naive datetime as UTC just because the machine happened to run in UTC.

## 7. UTC and fixed offsets

The `datetime` module provides `timezone` for fixed offsets from UTC and provides `timezone.utc` for UTC.

```python
import datetime as dt

utc_now = dt.datetime.now(dt.timezone.utc)
print(utc_now.tzinfo)
```

Python also provides `datetime.UTC` as an alias for the UTC timezone singleton in versions where that API is available.

For automation that needs named geographical time zones and daylight-saving rules, use `zoneinfo` rather than inventing fixed offsets.

## 8. IANA time zones with `zoneinfo`

The standard-library `zoneinfo` module provides IANA time-zone support.

```python
import datetime as dt
from zoneinfo import ZoneInfo

when = dt.datetime(2026, 10, 31, 18, 30, tzinfo=ZoneInfo("Asia/Kolkata"))
print(when)
```

Convert an aware datetime to another zone with `astimezone()`:

```python
utc_time = dt.datetime.now(dt.timezone.utc)
local_time = utc_time.astimezone(ZoneInfo("Asia/Kolkata"))
```

The IANA database contains time-zone rules that can account for changes such as daylight-saving transitions.

## 9. Time-zone data availability

`zoneinfo` normally uses system time-zone data. If that data is unavailable, it can fall back to the first-party `tzdata` package when installed.

If neither source contains the requested zone, `ZoneInfo` raises `ZoneInfoNotFoundError`.

Repository design decision:

- Use UTC-aware datetimes for cross-system timestamps where UTC is the intended reference.
- Use `ZoneInfo` when a real geographical time zone is required.
- Treat time-zone data availability as a deployment consideration for portable automation.

## 10. `timedelta` and durations

A `timedelta` represents a duration.

```python
import datetime as dt

retention = dt.timedelta(days=30)
retry_window = dt.timedelta(minutes=5)
```

Datetime arithmetic is straightforward:

```python
created = dt.datetime.now(dt.timezone.utc)
expires = created + dt.timedelta(hours=24)
```

You can subtract two compatible date/datetime values to obtain a `timedelta`:

```python
age = dt.datetime.now(dt.timezone.utc) - created
```

## 11. `timedelta.seconds` vs `total_seconds()`

A common mistake is assuming `.seconds` means the total number of seconds in a duration.

```python
import datetime as dt

duration = dt.timedelta(days=2, hours=3)

print(duration.seconds)        # seconds within the normalized day
print(duration.total_seconds()) # total duration in seconds
```

When the requirement is the total duration in seconds, use `total_seconds()`.

## 12. Parsing strings with `strptime()`

Use `datetime.strptime()` when you know the input format and need to parse a string into a datetime.

```python
import datetime as dt

text = "2026-10-31 18:30:00"
when = dt.datetime.strptime(text, "%Y-%m-%d %H:%M:%S")
print(when)
```

The format string describes the expected representation.

Common directives include:

```text
%Y  four-digit year
%m  month
%d  day
%H  hour (24-hour clock)
%M  minute
%S  second
```

Treat the input format as part of the interface contract. If the incoming timestamp format changes, parsing should fail visibly rather than silently producing an incorrect operational value.

## 13. Formatting with `strftime()`

Use `strftime()` to format a date/time object as text.

```python
import datetime as dt

when = dt.datetime(2026, 10, 31, 18, 30, 0)
formatted = when.strftime("%Y-%m-%dT%H:%M:%S")
print(formatted)
```

Formatting is presentation or serialization; it does not change the underlying datetime object.

## 14. ISO 8601 helpers

For machine-readable timestamps, the ISO-oriented methods are often preferable to inventing ad-hoc formats.

```python
import datetime as dt

when = dt.datetime.now(dt.timezone.utc)
text = when.isoformat()
restored = dt.datetime.fromisoformat(text)
```

Use the exact input/output contract expected by the system you are integrating with. Do not assume that every external system accepts every ISO representation.

## 15. Comparing timestamps

Use compatible date/time representations when comparing values.

For aware datetimes, compare values that have meaningful time-zone information:

```python
import datetime as dt

now = dt.datetime.now(dt.timezone.utc)
deadline = now + dt.timedelta(minutes=15)

if now < deadline:
    print("deadline has not passed")
```

Avoid mixing naive and aware datetimes in application logic. Establish one timestamp policy for the automation and follow it consistently.

## 16. Converting between time zones

`astimezone()` converts an aware datetime to another time zone.

```python
import datetime as dt
from zoneinfo import ZoneInfo

utc_time = dt.datetime.now(dt.timezone.utc)
india_time = utc_time.astimezone(ZoneInfo("Asia/Kolkata"))
new_york_time = utc_time.astimezone(ZoneInfo("America/New_York"))
```

The conversion represents the same instant using a different time-zone context.

## 17. DevOps use case: retention windows

Suppose an automation task should select records older than a defined retention period.

```python
import datetime as dt

now = dt.datetime.now(dt.timezone.utc)
cutoff = now - dt.timedelta(days=30)

if created_at < cutoff:
    print("candidate for retention processing")
```

Repository demonstration: the policy is intentionally simple. A production retention policy must follow the actual service's documented rules.

## 18. DevOps use case: deployment age

```python
import datetime as dt

started_at = dt.datetime(2026, 10, 31, 10, 0, tzinfo=dt.timezone.utc)
now = dt.datetime(2026, 10, 31, 12, 30, tzinfo=dt.timezone.utc)

age = now - started_at
print(age.total_seconds())
```

This pattern is useful for calculating elapsed time, provided both timestamps use compatible semantics.

## 19. DevOps use case: scheduling around a local time zone

If a job is defined as running at a local wall-clock time in a particular geographical zone, model that zone explicitly.

```python
import datetime as dt
from zoneinfo import ZoneInfo

scheduled = dt.datetime(
    2026, 10, 31, 18, 0,
    tzinfo=ZoneInfo("Asia/Kolkata"),
)

print(scheduled.astimezone(dt.timezone.utc))
```

This is different from simply attaching a fixed UTC offset because geographical time zones can have changing rules.

## 20. Common mistakes

### Mistake 1: Treating naive datetimes as UTC

A naive datetime does not inherently mean UTC. Make the time-zone policy explicit.

### Mistake 2: Mixing naive and aware values

Establish whether an automation uses naive or aware values and use compatible values in comparisons and arithmetic.

### Mistake 3: Using a fixed offset for a geographical zone

A fixed offset does not encode the complete rule set of an IANA geographical time zone. Use `ZoneInfo` when geographical time-zone behavior matters.

### Mistake 4: Using `.seconds` for a total duration

Use `total_seconds()` when total seconds are required.

### Mistake 5: Parsing without an explicit contract

Do not guess the input timestamp format. Verify the producer's format and parse accordingly.

### Mistake 6: Comparing formatted strings without understanding the format

Convert timestamps to appropriate date/time objects when performing date/time calculations or comparisons.

### Mistake 7: Assuming a timezone database is always available

`zoneinfo` depends on system time-zone data or the first-party `tzdata` package. Deployment environments should account for that dependency when needed.

## 21. Practical exercise: timestamp normalizer

Write a function that receives an ISO-formatted timestamp and returns its representation in UTC.

Requirements:

1. Parse the input.
2. Reject or clearly handle values that do not contain the required time-zone information.
3. Convert the value to UTC.
4. Return a machine-readable representation.
5. Test valid, invalid, and boundary inputs.

## 22. Practical exercise: retention calculator

Build a small script that:

1. accepts a creation timestamp;
2. accepts a retention duration in days;
3. calculates the cutoff timestamp;
4. reports whether the record is older than the cutoff.

Keep all operational timestamps in a consistent time-zone model.

## 23. Practical exercise: multi-region display

Create a script that starts from one UTC-aware timestamp and displays it in two geographical time zones using `ZoneInfo`.

Test it around a known daylight-saving transition and inspect the resulting offsets.

## 24. Knowledge check

Before moving on, you should be able to explain:

1. The difference between `date`, `time`, `datetime`, and `timedelta`.
2. The difference between naive and aware datetimes.
3. Why UTC is useful for cross-system timestamps.
4. When `timezone` is appropriate and when `ZoneInfo` is more appropriate.
5. How `strptime()` and `strftime()` differ.
6. When to use `isoformat()` and `fromisoformat()`.
7. Why `total_seconds()` differs from `.seconds`.
8. Why time-zone data availability matters in deployment environments.

## 25. Completion criteria

You have completed this topic when you can:

- create and compare dates and datetimes;
- calculate durations with `timedelta`;
- parse and format known timestamp formats;
- distinguish naive from aware values;
- create UTC-aware timestamps;
- convert timestamps with `ZoneInfo`;
- reason about retention and elapsed-time calculations;
- avoid common timezone and duration mistakes;
- test timestamp handling around boundaries and timezone transitions.

## 26. Verified facts vs repository design

### Verified facts

- Python's `datetime` module supplies `date`, `time`, `datetime`, `timedelta`, `tzinfo`, and `timezone` types.
- `datetime` objects can be naive or aware.
- `timedelta` represents a duration and provides `total_seconds()`.
- `datetime` provides parsing and formatting APIs such as `strptime()` and `strftime()`.
- `zoneinfo` provides IANA time-zone support and integrates with `datetime`.
- `zoneinfo` uses system time-zone data when available and can fall back to the first-party `tzdata` package.

### Repository design decisions

- The DevOps track favors explicit time-zone policy and UTC-aware timestamps for cross-system automation.
- `ZoneInfo` is introduced for geographical time-zone requirements.
- Exercises focus on retention, deployment age, scheduling, and timestamp normalization.

### Demonstrations

The timestamps, retention periods, service examples, and exercises in this file are teaching demonstrations created for this repository. They are not claims about any particular production system.

## 27. Authoritative references

- Python `datetime` — Basic date and time types: https://docs.python.org/3/library/datetime.html
- Python `zoneinfo` — IANA time zone support: https://docs.python.org/3/library/zoneinfo.html
- Python Standard Library: https://docs.python.org/3/library/
