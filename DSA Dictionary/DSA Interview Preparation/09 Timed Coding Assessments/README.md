# Timed Coding Assessments

> Practise solving algorithmic problems under a time limit while preserving correctness, clarity, and good engineering habits.

This section is part of the **DSA Interview Preparation** track. It provides structured practice sessions for Python coding interviews, including DevOps and SRE-flavoured problems.

These assessments are original repository exercises. Their durations, scoring rubrics, and difficulty labels are **practice-design choices**, not official interview standards or guarantees of performance in a real hiring process.

## Table of Contents

1. [How to use these assessments](#how-to-use-these-assessments)
2. [Rules for a realistic practice session](#rules-for-a-realistic-practice-session)
3. [Scoring rubric](#scoring-rubric)
4. [Assessment A — Foundations](#assessment-a--foundations)
5. [Assessment B — Intermediate patterns](#assessment-b--intermediate-patterns)
6. [Assessment C — DevOps and SRE scenarios](#assessment-c--devops-and-sre-scenarios)
7. [Review your performance](#review-your-performance)
8. [A repeatable practice schedule](#a-repeatable-practice-schedule)
9. [Common timed-interview mistakes](#common-timed-interview-mistakes)
10. [Readiness checklist](#readiness-checklist)
11. [References](#references)
12. [Navigation](#navigation)

---

## How to use these assessments

Complete the assessments in order:

| Assessment | Suggested time | Focus | Problems |
|---|---:|---|---:|
| A — Foundations | 30 minutes | Arrays, strings, sorting, edge cases | 2 |
| B — Intermediate patterns | 45 minutes | Sliding window, hash maps, deterministic ordering | 2 |
| C — DevOps and SRE scenarios | 60 minutes | Dependency graphs, time windows, operational contracts | 2 |

You do not need to finish every problem within the suggested time on your first attempt. The purpose is to measure your current process, find weak areas, and practise improving them.

### Recommended progression

1. Attempt a session without reading its solutions.
2. Record the time spent on each problem.
3. Run the provided tests and add your own edge cases.
4. Score your work using the rubric below.
5. Review the solutions and compare your reasoning.
6. Reattempt missed problems later without looking at the solution.

Do not treat a completed problem as mastered until you can explain the algorithm, its correctness, and its complexity without relying on the written answer.

## Rules for a realistic practice session

### Before the timer

- Use the Python version and editor you normally practise with.
- Prepare a local file or scratch project.
- Read the entire assessment instructions.
- Do not open the solution sections.
- Decide whether documentation is allowed for the session. Keep that rule consistent when comparing attempts.

### During the timer

For each problem:

1. Restate the input and output contract.
2. Ask about unclear assumptions rather than silently inventing them.
3. Write one normal example and at least two boundary examples.
4. Choose a data structure or algorithm and explain why it fits.
5. Implement the simplest correct approach.
6. Test the implementation before optimizing.
7. State time and auxiliary-space complexity.
8. If time remains, improve readability or investigate a more efficient approach.

If you get stuck, do not spend the entire session staring at the same code. Re-read the contract, reduce the input, trace the state, and identify the invariant that should hold.

### After the timer

- Stop coding when the time limit ends.
- Record what was complete and what remained.
- Run tests even if the solution is unfinished.
- Identify the earliest point where progress slowed.
- Review the solution only after recording your own approach.
- Add a regression test for every bug you discovered.

### Suggested time allocation

This is a practice guideline, not a fixed rule:

| Activity | Approximate share |
|---|---:|
| Understand contract and examples | 10–15% |
| Plan data structures and algorithm | 10–15% |
| Implement | 40–50% |
| Test and fix | 15–20% |
| Explain complexity and trade-offs | 5–10% |

Adjust the allocation when a problem is unfamiliar. Do not sacrifice contract understanding just to start typing sooner.

---

## Scoring rubric

Score each dimension from 0 to 4 for every problem.

| Dimension | 0 | 2 | 4 |
|---|---|---|---|
| Correctness | No working approach | Works for common cases, misses important cases | Correct for the stated contract and tested boundaries |
| Algorithm choice | No coherent approach | Correct but unnecessarily expensive or unclear | Suitable approach with justified complexity |
| Code quality | Difficult to follow or incomplete | Understandable with some avoidable issues | Clear names, simple control flow, appropriate structure |
| Testing | No meaningful tests | Typical case tested | Typical, boundary, empty, and adversarial cases tested |
| Explanation | Cannot explain approach | Partial reasoning or complexity | Clear invariant, correctness reasoning, and complexity |

**Maximum:** 20 points per problem.

Use the score diagnostically:
- Low correctness: revisit contracts, boundaries, and invariants.
- Low algorithm choice: revisit the pattern-selection guide.
- Low code quality: simplify control flow and improve naming.
- Low testing: practise constructing edge cases before coding.
- Low explanation: practise communicating the why, not only the what.

There is no universal passing threshold in this repository. Compare your own results across repeated attempts and look for consistent improvement.

---

## Assessment A — Foundations

<badge color="success">Suggested time: 30 minutes</badge>

**Focus:** arrays, strings, sorting, hash maps, and boundary handling.

Allocate approximately 15 minutes per problem. Try both before reading the solutions.

### Problem A1 — First unique character

**Difficulty:** Easy

**Suggested time:** 15 minutes

**Task**

Given a string, return the index of its first character that appears exactly once. Return -1 if no character is unique.

**Function signature**

~~~python
def first_unique_index(text: str) -> int:
    ...
~~~

**Examples**

| Input | Output | Reason |
|---|---:|---|
| "swiss" | 1 | "w" is the first character appearing once |
| "aabb" | -1 | Every character repeats |
| "z" | 0 | The only character is unique |
| "" | -1 | There is no character |

**Constraints and contract**

- Input is a Python string.
- Character equality follows Python string element equality.
- Return a zero-based index.
- Do not modify the input.

**What to explain**

- Why counting first and scanning second preserves the original order.
- The time and auxiliary-space complexity.
- How empty input and repeated characters are handled.

<details>
<summary>Solution and reasoning — open after your attempt</summary>

~~~python
from collections import Counter

def first_unique_index(text: str) -> int:
    counts = Counter(text)

    for index, character in enumerate(text):
        if counts[character] == 1:
            return index

    return -1
~~~

**Reasoning**

The first pass counts the frequency of each character. The second pass follows the original string order and returns the first position whose frequency is one. Since the second pass is in input order, the result is the first unique character, not merely any unique character.

**Complexity**

- Time: O(n) expected, where n is the string length.
- Auxiliary space: O(u), where u is the number of distinct characters.

The expected-time statement reflects dictionary-backed counting; it is not a claim that every possible dictionary operation has a strict worst-case O(1) bound.

**Tests**

~~~python
assert first_unique_index("swiss") == 1
assert first_unique_index("aabb") == -1
assert first_unique_index("z") == 0
assert first_unique_index("") == -1
assert first_unique_index("aabcc") == 2
~~~

</details>

### Problem A2 — Merge overlapping intervals

**Difficulty:** Easy to Medium

**Suggested time:** 15 minutes

**Task**

Given a list of intervals represented by [start, end], return a new list in which overlapping or touching intervals are merged.

**Function signature**

~~~python
def merge_intervals(intervals: list[list[int]]) -> list[list[int]]:
    ...
~~~

**Examples**

| Input | Output |
|---|---|
| [[1, 3], [2, 6], [8, 10]] | [[1, 6], [8, 10]] |
| [[1, 2], [2, 4]] | [[1, 4]] |
| [[5, 7]] | [[5, 7]] |
| [] | [] |

**Constraints and contract**

- Each interval contains exactly two integer endpoints.
- Every interval satisfies start <= end.
- Intervals are closed intervals.
- Touching intervals merge: [1, 2] and [2, 4] become [1, 4].
- Return a new list; do not mutate the outer input list or its interval lists.

**What to explain**

- Why sorting by start position is useful.
- Why comparing the next start with the current merged end is sufficient.
- The cost of sorting and the output storage.

<details>
<summary>Solution and reasoning — open after your attempt</summary>

~~~python
def merge_intervals(intervals: list[list[int]]) -> list[list[int]]:
    if not intervals:
        return []

    ordered = sorted((start, end) for start, end in intervals)
    merged = [[ordered[0][0], ordered[0][1]]]

    for start, end in ordered[1:]:
        current = merged[-1]

        if start <= current[1]:
            current[1] = max(current[1], end)
        else:
            merged.append([start, end])

    return merged
~~~

**Reasoning**

Sorting places intervals in nondecreasing start order. After sorting, any interval that overlaps the current merged interval must be considered before a later interval with a larger start. If the next start is at most the current end, extend the current end if necessary. Otherwise, the next interval begins a new merged range.

The comparison uses <= because the contract says touching closed intervals merge.

**Complexity**

- Time: O(n log n) for sorting, followed by O(n) scanning.
- Auxiliary space: O(n) for the sorted representation and merged output.

The implementation creates new interval pairs and does not modify the caller's lists.

**Tests**

~~~python
assert merge_intervals([]) == []
assert merge_intervals([[5, 7]]) == [[5, 7]]
assert merge_intervals([[1, 3], [2, 6], [8, 10]]) == [
    [1, 6], [8, 10]
]
assert merge_intervals([[1, 2], [2, 4]]) == [[1, 4]]
assert merge_intervals([[8, 9], [1, 3], [3, 5]]) == [[1, 5], [8, 9]]
~~~

</details>

---

## Assessment B — Intermediate patterns

<badge color="warning">Suggested time: 45 minutes</badge>

**Focus:** sliding windows, hash maps, deterministic ordering, and complexity analysis.

Allocate approximately 22 minutes per problem, leaving time to test and explain.

### Problem B1 — Longest substring without repeated characters

**Difficulty:** Medium

**Suggested time:** 22 minutes

**Task**

Given a string, return the length of its longest contiguous substring that contains no repeated characters.

**Function signature**

~~~python
def longest_unique_substring(text: str) -> int:
    ...
~~~

**Examples**

| Input | Output |
|---|---:|
| "abcabcbb" | 3 |
| "bbbbb" | 1 |
| "pwwkew" | 3 |
| "" | 0 |
| "abcdef" | 6 |

**Constraints and contract**

- Input is a Python string.
- A substring is contiguous.
- Character equality follows Python string element equality.
- Return a length, not the substring itself.

**What to explain**

- What the sliding window represents.
- Why the left boundary never moves backwards.
- Why the last-seen index must be checked against the current window.
- The time and auxiliary-space complexity.

<details>
<summary>Solution and reasoning — open after your attempt</summary>

~~~python
def longest_unique_substring(text: str) -> int:
    last_seen = {}
    left = 0
    best = 0

    for right, character in enumerate(text):
        previous = last_seen.get(character)

        if previous is not None and previous >= left:
            left = previous + 1

        last_seen[character] = right
        best = max(best, right - left + 1)

    return best
~~~

**Reasoning**

The current window is text[left:right + 1]. It contains no repeated characters.

When the current character has appeared inside the current window, move left to one position after its previous occurrence. The previous >= left condition prevents an occurrence from an older, already-excluded part of the string from moving the window backwards.

After adjusting the window, record the current index and update the best length.

**Complexity**

- Time: O(n) expected, where n is the string length.
- Auxiliary space: O(u), where u is the number of distinct characters in the input.

**Tests**

~~~python
assert longest_unique_substring("abcabcbb") == 3
assert longest_unique_substring("bbbbb") == 1
assert longest_unique_substring("pwwkew") == 3
assert longest_unique_substring("") == 0
assert longest_unique_substring("abcdef") == 6
assert longest_unique_substring("abba") == 2
~~~

</details>

### Problem B2 — Top K frequent values with deterministic ties

**Difficulty:** Medium

**Suggested time:** 22 minutes

**Task**

Given a list of integers and an integer k, return the k most frequent distinct values.

Order the result by:
1. Frequency descending.
2. Numeric value ascending when frequencies are equal.

**Function signature**

~~~python
def top_k_frequent(values: list[int], k: int) -> list[int]:
    ...
~~~

**Examples**

| Input | Output |
|---|---|
| values=[4, 4, 2, 2, 9], k=2 | [2, 4] |
| values=[7, 7, 7, 3, 3, 1], k=2 | [7, 3] |
| values=[5, 2, 5, 2], k=2 | [2, 5] |
| values=[], k=0 | [] |

**Constraints and contract**

- Values are integers.
- k must satisfy 0 <= k <= the number of distinct values.
- Raise ValueError if k is outside that range.
- Return each selected value once.
- The tie-breaking rule is part of the required output contract.

**What to explain**

- Why a frequency map is useful.
- How the sort key implements both ordering rules.
- The complexity in terms of n input values and u distinct values.
- Why deterministic tie-breaking is important for reproducible results.

<details>
<summary>Solution and reasoning — open after your attempt</summary>

~~~python
from collections import Counter

def top_k_frequent(values: list[int], k: int) -> list[int]:
    counts = Counter(values)

    if not 0 <= k <= len(counts):
        raise ValueError("k must be between zero and the number of distinct values")

    ordered = sorted(counts, key=lambda value: (-counts[value], value))
    return ordered[:k]
~~~

**Reasoning**

Counter builds a frequency map. Sorting the distinct keys with (-frequency, value) puts larger frequencies first and uses ascending numeric value as the tie-breaker. Slicing returns exactly the requested number of values.

**Complexity**

- Counting: O(n) expected.
- Sorting distinct values: O(u log u).
- Total: O(n + u log u) expected.
- Auxiliary space: O(u).

This solution intentionally sorts all distinct values for clarity. A heap or selection-based approach may be useful for other constraints, but should preserve the same deterministic tie-breaking contract.

**Tests**

~~~python
assert top_k_frequent([4, 4, 2, 2, 9], 2) == [2, 4]
assert top_k_frequent([7, 7, 7, 3, 3, 1], 2) == [7, 3]
assert top_k_frequent([5, 2, 5, 2], 2) == [2, 5]
assert top_k_frequent([], 0) == []

try:
    top_k_frequent([1, 1, 2], 3)
except ValueError:
    pass
else:
    raise AssertionError("Expected ValueError for k greater than distinct count")
~~~

</details>

---

## Assessment C — DevOps and SRE scenarios

<badge color="info">Suggested time: 60 minutes</badge>

**Focus:** dependency graphs, event windows, explicit contracts, and operational reasoning.

Allocate approximately 30 minutes per problem. Treat the scenarios as algorithm exercises, not as complete implementations of production systems.

### Problem C1 — Produce a valid service deployment order

**Difficulty:** Medium to Advanced

**Suggested time:** 30 minutes

**Scenario**

A deployment tool receives a set of services and dependency relationships. A service must be deployed only after all of its prerequisites have been deployed.

**Task**

Return a valid deployment order. If the dependencies contain a cycle, return None.

**Function signature**

~~~python
def deployment_order(
    services: list[str],
    dependencies: list[tuple[str, str]],
) -> list[str] | None:
    ...
~~~

Each dependency pair (prerequisite, dependent) means the prerequisite must appear before the dependent.

**Example**

~~~python
services = ["api", "worker", "database", "cache"]
dependencies = [
    ("database", "api"),
    ("cache", "api"),
    ("database", "worker"),
]
~~~

One valid output is:

~~~python
["cache", "database", "api", "worker"]
~~~

Another valid topological order may also be accepted. The reference solution below uses a min-heap to make the result deterministic for the same input set.

**Constraints and contract**

- Service names are unique strings.
- Every dependency endpoint must appear in services.
- Duplicate dependency pairs are ignored.
- A service with no dependencies is still included.
- If a dependency endpoint is unknown, raise ValueError.
- If a cycle exists, return None.
- When several services are ready at once, choose the lexicographically smallest available service.

**What to explain**

- Why this is a directed graph problem.
- What the in-degree represents.
- Why removing zero-in-degree nodes produces a topological order.
- How a cycle is detected.
- Why the min-heap adds a logarithmic factor.

<details>
<summary>Solution and reasoning — open after your attempt</summary>

~~~python
import heapq

def deployment_order(
    services: list[str],
    dependencies: list[tuple[str, str]],
) -> list[str] | None:
    if len(set(services)) != len(services):
        raise ValueError("service names must be unique")

    graph = {service: set() for service in services}
    indegree = {service: 0 for service in services}

    for prerequisite, dependent in dependencies:
        if prerequisite not in graph or dependent not in graph:
            raise ValueError("dependency references an unknown service")

        if dependent not in graph[prerequisite]:
            graph[prerequisite].add(dependent)
            indegree[dependent] += 1

    ready = [
        service
        for service in services
        if indegree[service] == 0
    ]
    heapq.heapify(ready)

    result = []

    while ready:
        service = heapq.heappop(ready)
        result.append(service)

        for dependent in graph[service]:
            indegree[dependent] -= 1
            if indegree[dependent] == 0:
                heapq.heappush(ready, dependent)

    if len(result) != len(services):
        return None

    return result
~~~

**Reasoning**

This is Kahn's topological sorting algorithm. The in-degree of a service is the number of prerequisite edges that have not yet been removed. Every zero-in-degree service is ready to be placed next.

When a service is placed in the result, its outgoing edges are removed by decrementing the in-degree of each dependent. A dependent becomes ready when its in-degree reaches zero.

If fewer services are processed than the number supplied, at least one cycle remains and no complete topological ordering exists.

The sets remove duplicate edges so that an identical dependency does not incorrectly increase the in-degree more than once. The min-heap chooses the lexicographically smallest currently available service.

**Complexity**

Let V be the number of services and E the number of distinct dependency edges.

- Building the graph: O(V + E) expected.
- Heap operations: O(V log V) in total.
- Processing edges: O(E), excluding heap insertions.
- Overall: O(E + V log V).
- Auxiliary space: O(V + E).

**Tests**

~~~python
assert deployment_order([], []) == []
assert deployment_order(["db"], []) == ["db"]

result = deployment_order(
    ["api", "worker", "database", "cache"],
    [("database", "api"), ("cache", "api"), ("database", "worker")],
)
assert result == ["cache", "database", "api", "worker"]

assert deployment_order(
    ["a", "b"],
    [("a", "b"), ("b", "a")],
) is None

assert deployment_order(
    ["a", "b"],
    [("a", "b"), ("a", "b")],
) == ["a", "b"]

try:
    deployment_order(["api"], [("missing", "api")])
except ValueError:
    pass
else:
    raise AssertionError("Expected ValueError for an unknown service")
~~~

</details>

### Problem C2 — Calculate a rolling failure rate

**Difficulty:** Medium

**Suggested time:** 30 minutes

**Scenario**

An alerting exercise receives timestamped event outcomes. Calculate the failure rate over a trailing time window.

**Task**

Given sorted events, the current time, and a window length, return failures_in_window divided by events_in_window.

Return 0.0 when there are no events in the window.

**Function signature**

~~~python
def rolling_failure_rate(
    events: list[tuple[int, bool]],
    now: int,
    window: int,
) -> float:
    ...
~~~

Each event is (timestamp, failed), where failed is True for a failure and False for a success.

**Example**

~~~python
events = [
    (90, False),
    (95, True),
    (97, False),
    (100, True),
]
now = 100
window = 10
~~~

The window is (90, 100]. The event at timestamp 90 is excluded. The remaining three events contain two failures, so the result is 2 / 3.

**Constraints and contract**

- Events are sorted by nondecreasing timestamp.
- Timestamps, now, and window are integers in the same time unit.
- window must be positive; otherwise raise ValueError.
- Include events satisfying now - window < timestamp <= now.
- Ignore future events.
- Return 0.0 if the window contains no events.
- The input list is not modified.

**What to explain**

- Why the interval is half-open.
- Why the lower boundary is strict.
- Why the denominator is the number of events in the window.
- How the implementation behaves when there are no events.

<details>
<summary>Solution and reasoning — open after your attempt</summary>

~~~python
def rolling_failure_rate(
    events: list[tuple[int, bool]],
    now: int,
    window: int,
) -> float:
    if window <= 0:
        raise ValueError("window must be positive")

    cutoff = now - window
    total = 0
    failures = 0

    for timestamp, failed in events:
        if timestamp > now:
            break

        if timestamp > cutoff:
            total += 1
            if failed:
                failures += 1

    if total == 0:
        return 0.0

    return failures / total
~~~

**Reasoning**

The contract defines the interval as (now - window, now]. Therefore an event exactly at the lower cutoff is excluded, while an event exactly at now is included.

Because events are sorted, the loop can stop when it reaches a future timestamp. Every event inside the window contributes one to the denominator; only failed events contribute to the numerator.

**Complexity**

- Time: O(n) worst case, where n is the number of events. The early break can reduce work when future events are present.
- Auxiliary space: O(1).

**Tests**

~~~python
assert rolling_failure_rate([], 100, 10) == 0.0

assert rolling_failure_rate(
    [(90, False), (95, True), (97, False), (100, True)],
    100,
    10,
) == 2 / 3

assert rolling_failure_rate(
    [(90, True), (100, False)],
    100,
    10,
) == 0.0

assert rolling_failure_rate(
    [(101, True), (102, False)],
    100,
    10,
) == 0.0

assert rolling_failure_rate(
    [(95, True), (95, False), (100, True)],
    100,
    10,
) == 2 / 3

try:
    rolling_failure_rate([], 100, 0)
except ValueError:
    pass
else:
    raise AssertionError("Expected ValueError for a non-positive window")
~~~

</details>

**Operational limitations**

This exercise is a batch algorithm over an already sorted in-memory list. It does not implement a production metrics query, streaming window, distributed aggregation, late-event policy, clock synchronization, or durable event processing. Those systems require additional contracts and design decisions.

---

## Review your performance

After each assessment, complete a short review. Be specific; “I need more practice” is not an actionable diagnosis.

| Question | Your notes |
|---|---|
| Which problem took the longest? | |
| Did I misunderstand any input or output contract? | |
| Which edge case did I miss initially? | |
| Did I choose the right data structure? | |
| Did I explain the invariant? | |
| Did I test before declaring completion? | |
| Was my complexity analysis accurate? | |
| What will I practise next? | |

### Classify every problem after review

Choose one status:

- **Solved independently:** correct implementation and explanation without hints.
- **Solved with a hint:** completed after receiving a meaningful hint.
- **Understood after review:** needed the solution or explanation.
- **Needs another attempt:** still cannot explain or implement the approach.

Do not count reading a solution as an independent solve.

### Track progress across attempts

| Date | Assessment | Correct problems | Hints used | Main weakness | Reattempt date |
|---|---|---:|---:|---|---|
| | A | | | | |
| | B | | | | |
| | C | | | | |

Use the same assessment again after a delay. A successful reattempt without reading the solution is stronger evidence of learning than immediate recognition after reviewing the answer.

---

## A repeatable practice schedule

Use this optional two-week cycle:

| Day | Practice |
|---|---|
| 1 | Assessment A under the time limit |
| 2 | Review A, write regression tests, revisit weak concepts |
| 3 | Reattempt A without solutions |
| 4 | Assessment B under the time limit |
| 5 | Review B and trace the sliding-window and frequency-map invariants |
| 6 | Reattempt B and explain complexity aloud |
| 7 | Rest or do a short revision session |
| 8 | Assessment C under the time limit |
| 9 | Review graph ordering and time-window boundaries |
| 10 | Reattempt C without solutions |
| 11 | Mix one easy and one medium problem from earlier sections |
| 12 | Practise explaining a solution while coding |
| 13 | Repeat the weakest assessment |
| 14 | Review progress and choose the next learning target |

Adjust the schedule to your available time. Consistent practice and honest review are more useful than completing many timed sessions without learning from mistakes.

---

## Common timed-interview mistakes

1. **Coding before understanding the contract.** Clarify boundaries, mutation, ordering, and invalid input first.
2. **Skipping examples.** A small hand-worked example often exposes a wrong assumption before implementation.
3. **Overengineering.** Start with the simplest correct algorithm that fits the constraints.
4. **Optimizing before testing.** Confirm baseline behavior before changing its complexity.
5. **Ignoring duplicate values.** Repeated values often reveal flaws in pointer, counting, and ordering logic.
6. **Using vague tie-breaking.** If several outputs are valid, state the rule; if the contract requires deterministic output, implement it explicitly.
7. **Forgetting complexity.** Include sorting, copying, hash maps, heaps, and output storage in the analysis.
8. **Not communicating.** Explain the plan, invariant, and trade-offs while working.
9. **Treating the timer as the only goal.** A fast incorrect answer is not a successful solution.
10. **Reading the solution too soon.** Record your own attempt first so the review reveals what you actually understand.

---

## Readiness checklist

- [ ] I can complete the foundations assessment and explain both solutions.
- [ ] I can use counting, sorting, and interval merging correctly.
- [ ] I can maintain a valid sliding-window invariant.
- [ ] I can use a frequency map and implement deterministic tie-breaking.
- [ ] I can model dependencies as a directed graph.
- [ ] I can explain topological sorting and cycle detection.
- [ ] I can define a time window precisely and test its endpoints.
- [ ] I can handle empty inputs and invalid arguments according to the contract.
- [ ] I can write tests before declaring a problem complete.
- [ ] I can explain time and auxiliary-space complexity.
- [ ] I can communicate my reasoning under a time limit.
- [ ] I can review mistakes and improve on a later attempt without reading the solution.

---

## References

The assessments are original repository exercises. The following official Python documentation supports the language and standard-library features used in the examples:

| Topic | Official reference |
|---|---|
| Unit testing, test cases, assertions, and test discovery | [unittest — Unit testing framework](https://docs.python.org/3/library/unittest.html) |
| Measuring small code sections | [timeit — Measure execution time of small code snippets](https://docs.python.org/3/library/timeit.html) |
| Time functions and performance counter | [time — Time access and conversions](https://docs.python.org/3/library/time.html) |
| Reproducible pseudo-random test data | [random — Generate pseudo-random numbers](https://docs.python.org/3/library/random.html) |
| Frequency counting | [collections — Container datatypes](https://docs.python.org/3/library/collections.html) |
| Heap operations | [heapq — Heap queue algorithm](https://docs.python.org/3/library/heapq.html) |

A seeded pseudo-random generator can help make generated practice inputs repeatable. Python's random module is not suitable for security-sensitive randomness; use the secrets module for security or cryptographic purposes.

---

## Navigation

- [← Previous: Debugging and Optimization](../08%20Debugging%20and%20Optimization/README.md)
- [↑ Interview Preparation Home](../README.md)
- [Interview Roadmap](../01%20Interview%20Roadmap/README.md)
- [Python Coding Interview](../02%20Python%20Coding%20Interview/README.md)
- [Pattern Recognition](../03%20Pattern%20Recognition/README.md)
- [Easy Problem Sets](../04%20Easy%20Problem%20Sets/README.md)
- [Medium Problem Sets](../05%20Medium%20Problem%20Sets/README.md)
- [Advanced Problem Sets](../06%20Advanced%20Problem%20Sets/README.md)
- [DevOps and SRE Problems](../07%20DevOps%20and%20SRE%20Problems/README.md)
- [Debugging and Optimization](../08%20Debugging%20and%20Optimization/README.md)

**Next:** Mock Interviews will be added in a later step.
