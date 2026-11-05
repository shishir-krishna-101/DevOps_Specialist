# 11 — Revision and Progress Tracking

A consistent practice record helps you answer four important questions:

1. What have I practised?
2. Which concepts can I solve independently?
3. Which mistakes keep recurring?
4. What should I revise next?

This guide provides a lightweight tracking system for the DSA Interview Preparation path. It is designed for Python coding interviews, including DevOps, SRE, platform, cloud, and general software engineering roles.

> **Practice-design note:** The tracking fields, review intervals, progress indicators, and readiness checks in this guide are repository recommendations. They are not scientifically validated learning intervals, official hiring criteria, or guarantees of interview performance. Adjust them to your schedule and the requirements of the roles you are targeting.

## Table of contents

1. [How to use this tracker](#1-how-to-use-this-tracker)
2. [What to record for each attempt](#2-what-to-record-for-each-attempt)
3. [Problem attempt log](#3-problem-attempt-log)
4. [How to measure progress](#4-how-to-measure-progress)
5. [Build a revision queue](#5-build-a-revision-queue)
6. [Weekly and monthly reviews](#6-weekly-and-monthly-reviews)
7. [Pattern and topic mastery](#7-pattern-and-topic-mastery)
8. [Interview readiness dashboard](#8-interview-readiness-dashboard)
9. [CSV tracking and data handling](#9-csv-tracking-and-data-handling)
10. [Common tracking mistakes](#10-common-tracking-mistakes)
11. [Templates](#11-templates)
12. [References and navigation](#12-references-and-navigation)

---

## 1. How to use this tracker

Use one record for every meaningful attempt at a problem. Do not overwrite an earlier attempt when you retry a problem. Keeping the history makes it possible to see whether you are improving, repeating the same mistake, or relying on hints.

### The practice loop

~~~text
Choose a problem
      |
      v
Attempt it without reading the solution
      |
      v
Record the result, time, hints, and tests
      |
      v
Review the explanation and identify gaps
      |
      v
Schedule a revision
      |
      v
Retry without notes
      |
      v
Test the pattern on an unfamiliar problem
~~~

### Recommended learning sequence

| Stage | What to do | Evidence to record |
|---|---|---|
| Learn | Study the concept and implement a small example | Concept notes and implementation |
| Practise | Solve a focused problem | Attempt record |
| Explain | Describe the invariant and complexity | Explanation quality |
| Test | Cover normal and edge cases | Test cases and outcomes |
| Revise | Retry after a delay without notes | Independent revision result |
| Transfer | Solve a new problem using the same pattern | Unfamiliar problem result |
| Simulate | Complete a timed assessment or mock | Scorecard and debrief |

A problem marked “solved” is not automatically mastered. You may have copied a solution, used a strong hint, or remembered the exact implementation. Record how the result was achieved.

### Start with a small system

You do not need a complicated dashboard or a new application. Start with:

- One spreadsheet or CSV file for attempts.
- One short list of concepts to revise.
- One weekly review.
- One monthly readiness check.

Add automation only when the manual process becomes difficult to maintain.

## 2. What to record for each attempt

The following fields form the suggested record format. They are repository design choices intended to make practice history useful and comparable.

| Field | Meaning | Example |
|---|---|---|
| date | Date of the attempt, ISO format | 2026-11-05 |
| problem_id | Stable identifier for the problem | EASY-001 |
| problem_title | Short problem name | First unique index |
| topic | Main DSA topic | Arrays and strings |
| pattern | Primary pattern used | Hash map |
| difficulty | Repository difficulty label | Easy |
| attempt | Type of attempt | first / revision / mock |
| result | Outcome | solved / partial / unsolved |
| independent | Solved without hints or solution? | yes / no |
| hint_level | Highest hint used, 0–4 | 0 |
| minutes | Time spent on this attempt | 18 |
| tests_passed | Number of tests passed, if executed | 8 |
| tests_total | Number of tests run | 8 |
| complexity_explained | Could you explain time and space costs? | yes / no |
| review_due | Planned next review date | 2026-11-08 |
| notes | Short mistake or learning note | Forgot empty input |

### Use consistent values

For this tracker, use these values consistently:

**Result**
- solved — the implementation satisfies the stated contract.
- partial — the approach or implementation is incomplete.
- unsolved — no correct solution was completed.

**Independent**
- yes — you derived and completed the solution without hints or reading the solution.
- no — you needed a hint, looked at the solution, or substantially copied an implementation.

**Hint level**
- 0 — no hint.
- 1 — clarification or example prompt.
- 2 — data-structure or modeling prompt.
- 3 — pattern-level hint.
- 4 — specific implementation guidance.

These definitions make the record easier to interpret. They are not a universal assessment scale.

### Separate attempts from problems

A problem can have several attempts:

| Date | Problem | Attempt | Result | Independent | Note |
|---|---|---|---|---|---|
| 2026-11-01 | EASY-001 | first | partial | no | Did not handle empty input |
| 2026-11-03 | EASY-001 | revision | solved | yes | Added empty and duplicate cases |
| 2026-11-10 | EASY-001 | revision | solved | yes | Rebuilt from memory |

Keep all three rows. The first attempt shows the original gap; later attempts show whether it was addressed.

## 3. Problem attempt log

### Spreadsheet layout

Create a sheet named **Attempts** and use the columns below. Keep one row per attempt.

~~~text
date,problem_id,problem_title,topic,pattern,difficulty,attempt,result,independent,hint_level,minutes,tests_passed,tests_total,complexity_explained,review_due,notes
2026-11-05,EASY-001,First unique index,Arrays and strings,Hash map,Easy,first,solved,yes,0,18,8,8,yes,2026-11-08,Checked empty and duplicate input
2026-11-06,MED-003,Longest unique substring,Strings,Sliding window,Medium,first,partial,no,2,32,5,7,no,2026-11-07,Moved left boundary backwards
~~~

The rows are examples of the repository's proposed format, not records of actual practice.

### Suggested spreadsheet validation

To reduce inconsistent data entry, use dropdowns or validation rules for:

- difficulty: Easy, Medium, Advanced
- attempt: first, revision, mock
- result: solved, partial, unsolved
- independent: yes, no
- complexity_explained: yes, no
- hint_level: whole number from 0 through 4

Use actual date cells where your spreadsheet supports them. If you exchange the data as CSV, store dates in the unambiguous YYYY-MM-DD form.

### Keep notes specific

Useful notes describe an observable issue:

- “Forgot to remove expired entries from the deque.”
- “Used the wrong edge direction in the dependency graph.”
- “Returned a value instead of its index.”
- “Did not test when the heap contains one item.”
- “Could not justify the expected-time hash-table assumption.”

Avoid notes such as “bad at DSA” or “need more practice.” They do not tell you what to do next.

## 4. How to measure progress

Use several indicators. No single metric captures problem-solving ability.

### A. Independent first-attempt solve rate

This measures how often a first attempt is solved without help.

For a selected review period:

~~~text
Independent first-attempt solve rate =
  first attempts marked solved
  AND independent = yes
  AND hint_level = 0
  ------------------------------------------------
  all first attempts with a recorded result
~~~

This is a practice indicator, not a hiring probability. Compare similar difficulty levels and problem types where possible.

### B. Revision recovery

Track problems that were initially partial or unsolved and later solved independently.

~~~text
Revision recovery =
  previously incomplete problems later solved independently
  -----------------------------------------------------------
  previously incomplete problems that received a revision attempt
~~~

Record the number of problems as well as the ratio. A ratio based on two problems is much less informative than the same ratio based on twenty.

### C. Explanation quality

After coding, check whether you can explain:

- The input and output contract.
- The selected data structure or algorithm.
- The invariant or key correctness argument.
- Time complexity and auxiliary space.
- Important edge cases.
- One alternative and its trade-off.

Use a simple status: clear, developing, or needs review. This is a self-review aid, not an objective score.

### D. Testing quality

Record whether you tested relevant cases, not only whether the example passed.

Depending on the problem, consider:

- Empty and one-element input.
- Duplicate values.
- Already sorted and reverse-sorted input.
- Negative and zero values.
- Missing keys or disconnected graph nodes.
- Boundary equality.
- Cycles and invalid input.
- Large input or a complexity-sensitive case.

A test count alone does not measure test quality. Eight trivial tests do not necessarily provide better coverage than four carefully chosen boundary tests.

### E. Time trend

Track time for repeated attempts at the same problem or for comparable problems of similar difficulty.

Do not compare raw times across unrelated problems. A shorter time is useful only when correctness, explanation, and testing remain strong.

### F. Pattern transfer

The strongest evidence that you understand a pattern is solving an unfamiliar problem that requires it. Track transfer attempts separately from revisions of a problem you have already seen.

### A compact weekly summary

| Indicator | This week | Previous week | What it tells you |
|---|---:|---:|---|
| First attempts |  |  | Practice volume |
| Independent first-attempt solves |  |  | Initial problem-solving success |
| Revisions completed |  |  | Follow-through |
| Problems recovered independently |  |  | Learning from earlier gaps |
| Timed sessions |  |  | Performance under time constraints |
| Patterns needing review |  |  | Next study priorities |

Fill this table with your own records. Do not set a target solely to increase the numbers.

## 5. Build a revision queue

A revision queue turns mistakes into scheduled work. It should prioritize unresolved gaps, not merely list every problem you have ever solved.

### Suggested review policy

The intervals below are a practical starting schedule created for this repository. They are not universal or scientifically guaranteed spacing intervals.

| Latest result | Suggested next action | Example review timing |
|---|---|---|
| Unsolved | Relearn the concept, then retry a smaller example | Next study session |
| Partial or solved with strong hints | Review the explanation, then retry independently | 1–3 days |
| Solved independently but explanation was weak | Practise explaining and testing it again | About 3–7 days |
| Solved independently with clear reasoning and tests | Revisit without notes | About 7–14 days |
| Repeatedly solved and transferred to a new problem | Move to mixed revision | About 14–21 days |

Adjust these intervals to your available study time and how well you retain the concept. If a revision fails, move the problem back to a nearer review slot.

### Priority levels

Use a small number of priority labels:

- **High:** Repeated errors, an important prerequisite, or a pattern that blocks several other topics.
- **Medium:** One known gap or a solution that still requires hints.
- **Low:** Previously solved independently and due for a maintenance review.

### Revision queue template

| Priority | Problem or concept | Why it is here | Next action | Review date | Status |
|---|---|---|---|---|---|
| High | Sliding window boundary | Moved left pointer incorrectly | Trace five windows by hand |  | Open |
| High | Topological sort | Missed cycle detection | Implement and test a cyclic graph |  | Open |
| Medium | Heap / top-K | Tie-breaking unclear | Solve a new top-K exercise |  | Open |
| Low | Hash-map counting | Previously solved independently | Explain expected-time assumption |  | Scheduled |

These are illustrative entries. Replace them with your own observed gaps.

### Rules for maintaining the queue

1. Add an item when an attempt reveals a specific weakness.
2. Write an action that can be completed and checked.
3. Set a review date that fits your schedule.
4. After revision, record a new attempt in the attempt log.
5. Close the queue item only when the required action is complete.
6. Reopen it if the same mistake appears again.

Do not copy every problem into the revision queue. Track problems in the attempt log; put only items requiring follow-up into the queue.

## 6. Weekly and monthly reviews

### Weekly review — 20 to 30 minutes

Once per week:

1. Review the attempt log.
2. Identify the two or three most common error types.
3. Check which revision items are due or overdue.
4. Select the next week's topics based on evidence.
5. Choose at least one unfamiliar problem for pattern transfer.
6. Schedule one timed assessment or mock interview if appropriate.
7. Write one concrete improvement goal.

### Weekly review worksheet

~~~text
Week beginning:

Problems attempted:
Problems solved independently:
Problems requiring hints:
Revisions completed:
Timed sessions completed:

Three recurring mistakes:
1.
2.
3.

Strongest pattern this week:
Pattern needing the most work:

One problem I can now explain clearly:
One unfamiliar problem I will attempt next:

Next week's specific goal:
~~~

### Monthly review — 30 to 45 minutes

At the end of the month:

- Compare independent first-attempt results with the previous month.
- Review patterns that repeatedly require hints.
- Revisit a selection of old problems without notes.
- Attempt one mixed set where the pattern is not announced.
- Review timed assessment and mock interview feedback.
- Decide whether to advance, repeat, or slow down a topic.
- Update the next month's study plan.

### Decide what to do next

| Evidence | Suggested decision |
|---|---|
| You cannot explain the basic operation | Return to the concept lesson |
| You can explain it but cannot implement it | Do a small implementation exercise |
| The code works only for examples | Focus on invariants and edge-case tests |
| You solve it with hints | Repeat a similar problem independently |
| You solve familiar problems but fail unfamiliar ones | Practise pattern recognition and transfer |
| You solve correctly but exceed the time allocation | Practise tracing, implementation fluency, and timed sets |
| You solve correctly but cannot explain complexity | Revisit complexity analysis and justify assumptions |
| You perform well in practice but struggle in mocks | Practise communication and interview workflow |

## 7. Pattern and topic mastery

Track patterns separately from individual problems. A single problem can involve several concepts, but choose one primary pattern for consistent reporting.

### Pattern review matrix

| Pattern | Can explain it? | Can implement it? | Can test edge cases? | Can recognize it in a new problem? |
|---|---|---|---|---|
| Hash map / set |  |  |  |  |
| Two pointers |  |  |  |  |
| Sliding window |  |  |  |  |
| Prefix sum |  |  |  |  |
| Stack / monotonic stack |  |  |  |  |
| Queue / deque |  |  |  |  |
| Binary search |  |  |  |  |
| Heap / top-K |  |  |  |  |
| Linked list pointers |  |  |  |  |
| Tree DFS / BFS |  |  |  |  |
| Graph DFS / BFS |  |  |  |  |
| Topological sort |  |  |  |  |
| Greedy algorithms |  |  |  |  |
| Backtracking |  |  |  |  |
| Dynamic programming |  |  |  |  |
| Disjoint Set Union |  |  |  |  |
| Intervals / sweep line |  |  |  |  |

Use these labels in each cell:

- **Not yet:** You have not practised this skill.
- **With help:** You need notes, hints, or a worked example.
- **Independent:** You can perform it without help on a familiar problem.
- **Transferred:** You can recognize and apply it to an unfamiliar problem.

Do not mark a pattern as transferred after repeating only the same memorized exercise. Use a new problem with a different story or input shape.

### Topic coverage checklist

- [ ] Python syntax and built-in containers.
- [ ] Arrays and strings.
- [ ] Hash maps and sets.
- [ ] Linked lists.
- [ ] Stacks and queues.
- [ ] Sorting and searching.
- [ ] Binary search.
- [ ] Trees and binary search trees.
- [ ] Heaps and priority queues.
- [ ] Graph traversal and topological sorting.
- [ ] Greedy algorithms.
- [ ] Backtracking.
- [ ] Dynamic programming.
- [ ] Advanced graph and data-structure topics.
- [ ] Complexity analysis.
- [ ] Testing and debugging.

This is a coverage checklist, not a requirement that every candidate master every advanced topic for every role. Prioritize using the job descriptions and interview expectations relevant to your target roles.

## 8. Interview readiness dashboard

Readiness is multidimensional. A strong result in one area does not compensate automatically for a major gap in another.

### Readiness dimensions

| Dimension | Evidence to collect |
|---|---|
| Python fluency | Can write and explain functions using appropriate built-in types |
| DSA foundations | Can select and implement core data structures |
| Pattern recognition | Can identify a suitable approach on unfamiliar problems |
| Correctness | Can explain why the algorithm works |
| Testing | Can create useful boundary and edge-case tests |
| Complexity | Can analyze time and auxiliary space with stated assumptions |
| Debugging | Can isolate a failing case and make a targeted correction |
| Communication | Can explain decisions and respond to follow-ups |
| Timed performance | Can preserve correctness within a practice time limit |
| Role relevance | Can connect algorithmic reasoning to operational scenarios where appropriate |

### Self-assessment

For each dimension, choose one status:

- **Not assessed**
- **Needs work**
- **Developing**
- **Consistent in practice**

Use “consistent in practice” only when you have repeated evidence across more than one problem or session. This is a self-assessment label, not a hiring prediction.

### Readiness review questions

Before considering yourself ready to start real interview practice, ask:

1. Can I solve an unfamiliar easy problem without looking up the pattern?
2. Can I make progress on medium problems and explain where I get stuck?
3. Can I ask clarifying questions before coding?
4. Can I explain an invariant or correctness argument?
5. Can I test more than the supplied examples?
6. Can I state time and auxiliary-space complexity accurately?
7. Can I debug a failing implementation without randomly changing code?
8. Can I complete a mock interview and use feedback constructively?
9. Have I practised the language and tools expected by the roles I am applying for?
10. Can I explain when an algorithmic exercise is too simplified to represent a production system?

If several answers are “no,” use the relevant sections of this track to guide your next practice cycle. You do not need to wait for a perfect score before applying or gaining real interview experience.

## 9. CSV tracking and data handling

A spreadsheet is usually enough. CSV is useful when you want a plain-text log that can be read by spreadsheet tools or processed with Python.

### Recommended CSV practices

- Keep a header row.
- Use one row per attempt.
- Use consistent field names and categorical values.
- Use ISO-formatted dates such as YYYY-MM-DD.
- Quote or escape fields containing commas, quotes, or line breaks using a CSV-aware tool.
- Keep notes concise; store longer explanations in a separate notes document.
- Back up the file before making bulk edits.
- Do not store confidential interview questions or personal information about interviewers.

The Python csv module provides reader and writer interfaces, including DictReader and DictWriter for dictionary-shaped rows. CSV dialect differences exist between applications, so do not assume every CSV producer uses identical formatting.

### Optional Python example: read and summarize a CSV log

Save the following example as progress_report.py in the same directory as progress.csv. It reports attempt counts by pattern and does not modify the input file.

~~~python
import csv
from collections import Counter
from pathlib import Path


CSV_PATH = Path("progress.csv")


def summarize_progress(path: Path) -> None:
    required = {"problem_id", "pattern", "attempt", "result", "independent"}

    with path.open("r", encoding="utf-8", newline="") as csv_file:
        reader = csv.DictReader(csv_file)

        if reader.fieldnames is None:
            raise ValueError("CSV file is missing a header row")

        missing = required - set(reader.fieldnames)
        if missing:
            raise ValueError(f"Missing required columns: {sorted(missing)}")

        total_attempts = 0
        results = Counter()
        attempts_by_pattern = Counter()
        independent_first_solves = 0
        first_attempts = 0

        for line_number, row in enumerate(reader, start=2):
            total_attempts += 1

            result = row["result"].strip().lower()
            attempt = row["attempt"].strip().lower()
            independent = row["independent"].strip().lower()
            pattern = row["pattern"].strip() or "Unspecified"

            if result not in {"solved", "partial", "unsolved"}:
                raise ValueError(
                    f"Invalid result on CSV line {line_number}: {result!r}"
                )

            if attempt not in {"first", "revision", "mock"}:
                raise ValueError(
                    f"Invalid attempt type on CSV line {line_number}: {attempt!r}"
                )

            if independent not in {"yes", "no"}:
                raise ValueError(
                    f"Invalid independent value on CSV line {line_number}: "
                    f"{independent!r}"
                )

            results[result] += 1
            attempts_by_pattern[pattern] += 1

            if attempt == "first":
                first_attempts += 1
                if result == "solved" and independent == "yes":
                    independent_first_solves += 1

    print(f"Total attempts: {total_attempts}")
    print(f"Results: {dict(results)}")
    print(f"First attempts: {first_attempts}")
    print(f"Independent first-attempt solves: {independent_first_solves}")

    if first_attempts:
        rate = independent_first_solves / first_attempts
        print(f"Independent first-attempt solve rate: {rate:.1%}")

    print("Attempts by pattern:")
    for pattern, count in attempts_by_pattern.most_common():
        print(f"  {pattern}: {count}")


if __name__ == "__main__":
    summarize_progress(CSV_PATH)
~~~

Run it from the directory containing both files:

~~~bash
python progress_report.py
~~~

This example assumes the categorical values defined earlier in this guide. It counts attempts, not unique problems. If the same problem appears in multiple rows, each row contributes one attempt. The independent first-attempt rate uses first-attempt rows as its denominator; it should not be interpreted as a probability of passing an interview.

### JSON as an alternative

JSON can be useful when you want to store nested information, such as several test results or a list of revision notes for one problem. The Python json module provides encoding and decoding functions.

For a simple flat attempt log, CSV is usually easier to inspect and edit. Choose one canonical format rather than maintaining two unsynchronized copies.

Do not load untrusted JSON without considering input size and resource usage. The official documentation warns that malicious JSON input can consume substantial CPU and memory.

### Dates and time zones

For a date-only study log, an ISO date is usually sufficient. If you record timestamps across devices or time zones, define whether the value is local time or UTC and use timezone-aware datetimes when a specific moment must be unambiguous. Avoid mixing naive and timezone-aware timestamps in the same calculation.

### Summaries and averages

The statistics module provides basic descriptive calculations such as mean and median. Use these only when the data is comparable. For example, median solve time across unrelated easy and advanced problems can be misleading. Compare repeated attempts at the same problem or similar problems at the same difficulty, and document excluded or missing values.

## 10. Common tracking mistakes

### Counting solved problems instead of measuring independent ability

A high solved count can include copied solutions, heavily hinted attempts, and repeated memorized problems. Track independence and transfer separately.

### Overwriting previous attempts

Replacing an old result hides the learning process. Append a new row for each attempt.

### Setting arbitrary daily targets

A target such as “solve ten problems every day” may reward rushing. Set a sustainable schedule that includes explanation, testing, and revision.

### Treating all problems as equally difficult

A short array exercise and an advanced graph problem should not be compared only by completion time. Group comparisons by topic, pattern, and difficulty.

### Confusing test count with test quality

A large number of repetitive tests does not guarantee good coverage. Record the edge cases that matter for the problem contract.

### Keeping a revision list without dates or actions

A list of topics is not a revision system. Every queued item should have a next action and a review date.

### Ignoring repeated mistakes

If the same boundary error appears in several problems, revise the underlying concept rather than memorizing fixes for each problem.

### Treating a dashboard as a hiring prediction

Practice data can show where you are improving and where you need work. It cannot reliably predict the result of every company's interview process.

## 11. Templates

### Daily practice log

~~~text
Date:

Problem ID and title:
Topic:
Pattern:
Difficulty:
Attempt type: first / revision / mock

Result: solved / partial / unsolved
Independent: yes / no
Hint level: 0 / 1 / 2 / 3 / 4
Time spent:
Tests passed / tests run:

Could I explain the invariant?
Could I explain time and auxiliary space?
Which edge case did I test?
What mistake did I make?
What did I learn?

Next action:
Review due:
~~~

### Weekly review

~~~text
Week beginning:

Total attempts:
Independent first-attempt solves:
Revisions completed:
Unresolved problems:
Timed assessments or mocks:

Strongest topic:
Weakest topic:
Most frequent mistake:
Pattern that needs another example:

One problem to revisit:
One unfamiliar problem to attempt:
One specific goal for next week:
~~~

### Monthly review

~~~text
Month:

Topics covered:
Patterns transferred to unfamiliar problems:
Patterns still requiring hints:
Repeated bugs or missed edge cases:
Timed practice completed:
Mock interview feedback:

What improved:
What did not improve:
What should be removed from the plan:
What should receive more time:

Next month's top three priorities:
1.
2.
3.
~~~

### Personal revision commitments

~~~text
I will review this item:
I will complete this action:
I will consider it complete when:
I will record the result in:
~~~

## 12. References and navigation

### Official Python documentation

- [csv — CSV file reading and writing](https://docs.python.org/3/library/csv.html) — reader, writer, DictReader, DictWriter, and CSV dialect considerations.
- [json — JSON encoder and decoder](https://docs.python.org/3/library/json.html) — serialization and deserialization of supported Python data.
- [datetime — Basic date and time types](https://docs.python.org/3/library/datetime.html) — date/time arithmetic and aware versus naive objects.
- [statistics — Mathematical statistics functions](https://docs.python.org/3/library/statistics.html) — descriptive statistics and data limitations.
- [pathlib — Object-oriented filesystem paths](https://docs.python.org/3/library/pathlib.html) — filesystem path handling.
- [unittest — Unit testing framework](https://docs.python.org/3/library/unittest.html) — repeatable test cases, fixtures, suites, and runners.

These references support Python standard-library behavior. The tracker format, revision policy, example data, and readiness dimensions are repository design decisions.

### Continue the DSA Interview Preparation path

- [DSA Interview Preparation — Track overview](../README.md)
- [01 — Interview Roadmap](../01%20Interview%20Roadmap/README.md)
- [02 — Python Coding Interview](../02%20Python%20Coding%20Interview/README.md)
- [03 — Pattern Recognition](../03%20Pattern%20Recognition/README.md)
- [04 — Easy Problem Sets](../04%20Easy%20Problem%20Sets/README.md)
- [05 — Medium Problem Sets](../05%20Medium%20Problem%20Sets/README.md)
- [06 — Advanced Problem Sets](../06%20Advanced%20Problem%20Sets/README.md)
- [07 — DevOps and SRE Problems](../07%20DevOps%20and%20SRE%20Problems/README.md)
- [08 — Debugging and Optimization](../08%20Debugging%20and%20Optimization/README.md)
- [09 — Timed Coding Assessments](../09%20Timed%20Coding%20Assessments/README.md)
- [10 — Mock Interviews](../10%20Mock%20Interviews/README.md)
