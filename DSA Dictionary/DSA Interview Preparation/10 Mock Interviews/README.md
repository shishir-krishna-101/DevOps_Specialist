# 10 — Mock Interviews

Practise the complete interview experience: clarify an unfamiliar problem, explain an approach, write correct Python, test it, analyze complexity, and respond to follow-up questions.

This guide is designed for Python coding interviews across software engineering, DevOps, SRE, platform engineering, and cloud roles.

> **Practice-design note:** The interview durations, prompts, role instructions, and scorecard in this file are original repository exercises. They are not official hiring standards, predictions of a particular employer's process, or guarantees of interview performance. Real interview formats vary by organization, role, level, and interviewer.

## Table of contents

1. [How to use this guide](#1-how-to-use-this-guide)
2. [Mock interview ground rules](#2-mock-interview-ground-rules)
3. [A repeatable interview workflow](#3-a-repeatable-interview-workflow)
4. [Mock 1 — Frequency and first unique item](#4-mock-1--frequency-and-first-unique-item)
5. [Mock 2 — Longest substring without repetition](#5-mock-2--longest-substring-without-repetition)
6. [Mock 3 — Service deployment order](#6-mock-3--service-deployment-order)
7. [Mock 4 — Rolling failure rate](#7-mock-4--rolling-failure-rate)
8. [Mock 5 — Debug a retry scheduler](#8-mock-5--debug-a-retry-scheduler)
9. [Mock 6 — Python code review and trade-offs](#9-mock-6--python-code-review-and-trade-offs)
10. [Interviewer and candidate instructions](#10-interviewer-and-candidate-instructions)
11. [Evaluation scorecard](#11-evaluation-scorecard)
12. [Follow-up question bank](#12-follow-up-question-bank)
13. [Debrief and improvement plan](#13-debrief-and-improvement-plan)
14. [Solo and remote practice](#14-solo-and-remote-practice)
15. [Readiness checklist](#15-readiness-checklist)
16. [References and navigation](#16-references-and-navigation)

---

## 1. How to use this guide

Complete the mocks in order the first time. Later, repeat them in shuffled order so that you practise recognizing a problem pattern rather than memorizing a sequence.

Each mock contains:

- **Candidate prompt:** The information the interviewer may share initially.
- **Clarifications to ask:** Questions a careful candidate should consider.
- **Practice contract:** Explicit assumptions used by the reference solution.
- **Interviewer guide:** Hints, expected reasoning, and follow-up prompts. A partner should keep this section hidden from the candidate.
- **Reference solution:** One valid implementation, not the only acceptable implementation.
- **Tests and complexity:** Examples to verify and a reasoned cost analysis.
- **Debrief:** What to review after the session.

For a realistic attempt, read only the candidate prompt first. Do not inspect the reference solution or interviewer guide until the session ends.

### Suggested progression

| Mock | Main skill | Practice time |
|---|---|---:|
| 1. First unique item | Hash map / counting | 25 minutes |
| 2. Longest unique substring | Sliding window | 35 minutes |
| 3. Service deployment order | Graph / topological sort | 40 minutes |
| 4. Rolling failure rate | Time window / queue | 35 minutes |
| 5. Retry scheduler debugging | Heap / debugging | 40 minutes |
| 6. Python code review | Correctness / maintainability | 30 minutes |

The durations are suggested practice allocations, not external benchmarks. You may extend them while learning; later, repeat the same problem within the suggested time.

## 2. Mock interview ground rules

### Candidate rules

1. Restate the problem in your own words.
2. Ask clarifying questions before committing to assumptions.
3. Describe a straightforward approach before optimizing it.
4. State the key data structure and why it fits.
5. Write executable code rather than pseudocode unless asked otherwise.
6. Test a normal case, a boundary case, and an edge case.
7. Explain time and auxiliary-space complexity.
8. Discuss limitations and trade-offs honestly.
9. If stuck, explain what you know and try a smaller example.
10. Do not claim code was tested unless you actually ran it.

### Interviewer rules

- Give the candidate the prompt, not the solution.
- Answer clarification questions consistently with the practice contract.
- Allow thinking time. Do not turn the session into a tutorial.
- Use hints only when needed; record which hints were required.
- Ask the candidate to trace code and explain complexity.
- Evaluate reasoning and correctness separately.
- Give specific, behavior-based feedback after the session.

### What a mock is not

A mock interview is a learning exercise, not a hiring decision. The scorecard below is a way to make feedback repeatable. It is not a validated assessment instrument and should not be used to infer a candidate's ability from one session.

## 3. A repeatable interview workflow

Use this sequence in every mock:

| Stage | Candidate action | What good practice looks like |
|---|---|---|
| Understand | Restate the task | Identifies input, output, and required behavior |
| Clarify | Ask about constraints | Resolves duplicates, empty input, ordering, invalid values, and boundaries |
| Model | Work through an example | Shows how the state changes |
| Design | Explain an approach | Connects the data structure to the required operations |
| Implement | Write readable code | Uses clear names and keeps responsibilities small |
| Verify | Trace and test | Checks ordinary, boundary, and adversarial cases |
| Analyze | Explain complexity | Defines variables and states assumptions |
| Extend | Answer a follow-up | Adapts the design without losing correctness |

### Useful phrases

- “I want to confirm whether the input can be empty.”
- “I will preserve the original order unless you want sorted output.”
- “The invariant of this window is that it contains no repeated characters.”
- “This operation is expected constant time under the usual hash-table assumptions.”
- “I have not benchmarked this; the complexity analysis is theoretical.”
- “This is an in-memory exercise. A distributed production implementation would need additional coordination and failure handling.”

These phrases are prompts for clear communication, not scripts to recite.

---

## 4. Mock 1 — Frequency and first unique item

<badge color="success">Foundational</badge>

### Candidate prompt

Given a list of integers, return the index of the first value that appears exactly once. Return -1 if no such value exists.

Example:

~~~python
values = [4, 5, 1, 2, 1, 4, 5]
# Expected result: 3
~~~

The value at index 3 is 2, and it occurs once.

### Clarifications to ask

- Can the list be empty? Yes.
- Can values be negative or repeated? Yes.
- Should the answer be a value or an index? An index.
- Does “first” mean the smallest value? No. It means the earliest position in the original list.
- May the input be modified? Prefer not to modify it.

### Practice contract

- Input is a list of integers.
- Return the zero-based index of the first value whose total frequency is one.
- Return -1 for empty input or when every value repeats.
- Preserve the input list.

### Interviewer guide

**Expected approach:** Count each value, then scan the original list from left to right. The first value with count one determines the answer.

**Why two passes?** The first pass establishes total frequencies. The second pass preserves the original order while locating the first unique value.

**Hint 1:** What information would let you decide whether a value is unique?

**Hint 2:** Can you calculate frequencies once and then use them in a second scan?

### Reference solution

~~~python
from collections import Counter


def first_unique_index(values: list[int]) -> int:
    counts = Counter(values)

    for index, value in enumerate(values):
        if counts[value] == 1:
            return index

    return -1


assert first_unique_index([4, 5, 1, 2, 1, 4, 5]) == 3
assert first_unique_index([7, 7, 8, 8]) == -1
assert first_unique_index([]) == -1
assert first_unique_index([-1, 0, -1]) == 1
~~~

### Correctness reasoning

Counter stores the total frequency of each value. The second loop visits indices in increasing order. It returns only when the current value has total frequency one. Therefore, no earlier position can contain a qualifying value, and the returned index is the first unique position. If no position qualifies, the function returns -1.

### Complexity

Let n be the number of values and u the number of distinct values.

- **Time:** O(n) expected for counting and the second scan, assuming expected constant-time dictionary operations.
- **Auxiliary space:** O(u) for the frequency table.

### Follow-up questions

1. Return the first unique value instead of its index. What changes?
2. Return all unique indices. What is the output-size cost?
3. What if values arrive as a stream and you cannot store the entire input?
4. What if the input contains unhashable objects?

### Debrief

- Did the candidate clarify index versus value?
- Did they preserve original order?
- Did they avoid sorting, which would change positional meaning?
- Did they test empty input and all-duplicate input?

---

## 5. Mock 2 — Longest substring without repetition

<badge color="info">Intermediate</badge>

### Candidate prompt

Given a string, return the length of the longest contiguous substring that contains no repeated characters.

Examples:

~~~python
assert longest_unique_substring("abcabcbb") == 3
assert longest_unique_substring("bbbbb") == 1
assert longest_unique_substring("pwwkew") == 3
assert longest_unique_substring("") == 0
~~~

For the third example, “wke” is a valid substring of length 3. “pwke” is not a substring because the characters are not contiguous.

### Clarifications to ask

- Is the result a substring or its length? Its length.
- Must characters be contiguous? Yes.
- Is the comparison case-sensitive? Yes, for this exercise.
- Can the input be empty? Yes.
- Are spaces and punctuation characters included? Yes; every character is treated as an ordinary character.

### Practice contract

- Input is a Python string.
- Return the maximum length of a contiguous substring with all distinct characters.
- Empty input returns zero.
- Character equality follows Python string equality.

### Interviewer guide

**Expected pattern:** Sliding window with a dictionary mapping each character to its most recent index.

**Invariant:** The current window from left through right contains no repeated characters.

When a repeated character appears inside the current window, move the left boundary just beyond its previous occurrence. Do not move the left boundary backwards.

**Hint 1:** Can you maintain a window that is always valid?

**Hint 2:** Store the last index at which each character was seen.

### Reference solution

~~~python
def longest_unique_substring(text: str) -> int:
    last_seen: dict[str, int] = {}
    left = 0
    best = 0

    for right, char in enumerate(text):
        previous = last_seen.get(char)

        if previous is not None and previous >= left:
            left = previous + 1

        last_seen[char] = right
        best = max(best, right - left + 1)

    return best


assert longest_unique_substring("abcabcbb") == 3
assert longest_unique_substring("bbbbb") == 1
assert longest_unique_substring("pwwkew") == 3
assert longest_unique_substring("") == 0
assert longest_unique_substring("abba") == 2
assert longest_unique_substring("a b!") == 4
~~~

### Trace the invariant

For “abba”:

| Right index | Character | Previous index | Left after update | Current window | Best |
|---:|---|---:|---:|---|---:|
| 0 | a | None | 0 | a | 1 |
| 1 | b | None | 0 | ab | 2 |
| 2 | b | 1 | 2 | b | 2 |
| 3 | a | 0 | 2 | ba | 2 |

At the final character, the previous “a” is outside the active window. The left boundary must not move backwards.

### Correctness reasoning

Before processing each right index, the active window begins at left. If the current character was last seen at or after left, moving left to one position after that occurrence removes the duplicate. If its previous occurrence is before left, the current window remains valid without moving left. Thus, after the update, the window is valid. Every valid window ending at right is no longer than the maintained window, so updating best with its length considers the best candidate for that endpoint. Taking the maximum across endpoints yields the answer.

### Complexity

- **Time:** O(n) expected, because each character is processed once and dictionary operations are expected O(1).
- **Auxiliary space:** O(min(n, k)) for the distinct characters stored, where k is the size of the character domain represented in the input.

### Follow-up questions

1. Return the actual substring as well as its length.
2. Return the earliest longest substring when multiple answers tie.
3. How would you adapt the definition if comparisons were case-insensitive?
4. What changes if the input is a stream and only a bounded window can be retained?

### Debrief

- Did the candidate articulate the window invariant?
- Did they guard against moving left backwards?
- Did they distinguish a substring from a subsequence?
- Did they test a repeated character outside the active window?

---

## 6. Mock 3 — Service deployment order

<badge color="warning">Advanced</badge>

### Candidate prompt

A deployment system has services and dependency relationships. If service B depends on service A, A must be deployed before B.

Given a list of service names and dependency pairs, return a valid deployment order. If the dependencies contain a cycle, report that no valid order exists.

Example:

~~~python
services = ["api", "worker", "database", "cache"]
dependencies = [
    ("database", "api"),
    ("cache", "api"),
    ("api", "worker"),
]
# One valid result: database, cache, api, worker
~~~

Each pair is written as (prerequisite, dependent).

### Clarifications to ask

- Can a service have multiple prerequisites? Yes.
- Can services be disconnected? Yes.
- Can there be a cycle? Yes; detect it.
- Can a dependency mention an unknown service? No for this exercise; reject invalid input.
- Is one exact order required? No. Any valid topological order is acceptable.
- Should ties be deterministic? Yes. Use lexicographically smallest available service first.

### Practice contract

- Service names are unique strings.
- Each dependency is a pair (prerequisite, dependent).
- Every referenced service must exist in services.
- Duplicate dependency pairs are ignored.
- Return a lexicographically smallest available topological order.
- Return None if a cycle exists.
- Raise ValueError for duplicate service names or unknown dependency endpoints.

### Interviewer guide

**Expected pattern:** Kahn's topological sort algorithm using an indegree map and a min-heap.

The indegree of a node is the number of incoming dependency edges. Start with every node whose indegree is zero. Remove the lexicographically smallest available node, append it to the result, and decrement the indegree of its dependents. Newly zero-indegree nodes become available.

If fewer nodes are emitted than the number of services, at least one cycle remains.

**Hint 1:** What does it mean for a service to be ready to deploy?

**Hint 2:** How can you efficiently choose the lexicographically smallest ready service?

### Reference solution

~~~python
import heapq


def deployment_order(
    services: list[str],
    dependencies: list[tuple[str, str]],
) -> list[str] | None:
    if len(set(services)) != len(services):
        raise ValueError("service names must be unique")

    service_set = set(services)
    graph: dict[str, list[str]] = {service: [] for service in services}
    indegree = {service: 0 for service in services}
    seen_edges: set[tuple[str, str]] = set()

    for prerequisite, dependent in dependencies:
        if prerequisite not in service_set or dependent not in service_set:
            raise ValueError("dependency references an unknown service")

        edge = (prerequisite, dependent)
        if edge in seen_edges:
            continue

        seen_edges.add(edge)
        graph[prerequisite].append(dependent)
        indegree[dependent] += 1

    ready = [
        service for service in services
        if indegree[service] == 0
    ]
    heapq.heapify(ready)

    order: list[str] = []

    while ready:
        service = heapq.heappop(ready)
        order.append(service)

        for dependent in graph[service]:
            indegree[dependent] -= 1
            if indegree[dependent] == 0:
                heapq.heappush(ready, dependent)

    if len(order) != len(services):
        return None

    return order


assert deployment_order(
    ["api", "worker", "database", "cache"],
    [("database", "api"), ("cache", "api"), ("api", "worker")],
) == ["cache", "database", "api", "worker"]

assert deployment_order(["a", "b"], [("a", "b"), ("b", "a")]) is None
assert deployment_order(["a", "b"], []) == ["a", "b"]
assert deployment_order([], []) == []
assert deployment_order(["a", "b"], [("a", "b"), ("a", "b")]) == ["a", "b"]
~~~

### Correctness reasoning

A service is added to ready only when its indegree reaches zero, meaning all its prerequisites have already been emitted. Therefore, emitting it respects every incoming dependency. Each emitted service has its outgoing edges processed once, reducing the indegree of each dependent. If all services are emitted, every dependency was respected. If some services remain, none of the remaining nodes can become ready; the remaining dependency subgraph contains a cycle.

The min-heap ensures that the smallest currently available service is selected at each step, giving the specified deterministic tie-breaking rule.

### Complexity

Let V be the number of services and E the number of distinct dependency pairs.

- **Time:** O(V + E + V log V) = O(E + V log V). Each edge is processed once, and each service enters and leaves the heap at most once.
- **Auxiliary space:** O(V + E) for the graph, indegrees, edge set, heap, and result.

### Follow-up questions

1. Return any valid order without lexicographic tie-breaking. Which data structure could replace the heap?
2. Return the actual cycle path rather than only reporting that a cycle exists.
3. How would you prioritize services by deployment risk?
4. What additional concerns arise when deployments execute in parallel?
5. Why is a valid topological order not, by itself, a complete production deployment plan?

### Production-context discussion

This function is a graph-algorithm exercise. It does not perform deployments, check service health, manage rollback, coordinate concurrent workers, or handle partial failures. A production deployment orchestrator requires additional state management and operational safeguards.

### Debrief

- Did the candidate define the dependency edge direction correctly?
- Did they detect cycles?
- Did they handle duplicate edges?
- Did they distinguish a valid order from a production rollout strategy?

---

## 7. Mock 4 — Rolling failure rate

<badge color="info">DevOps / SRE</badge>

### Candidate prompt

You receive timestamped request outcomes in non-decreasing timestamp order. Each outcome is either successful or failed.

For every event, calculate the failure rate among events in the trailing time window of width W that ends at the current event timestamp.

For this exercise, the window is **left-open and right-closed**: (current_timestamp - W, current_timestamp]. Return one rate for every input event.

Example:

~~~python
events = [
    (1, False),
    (2, True),
    (4, True),
    (5, False),
]
# With W = 3:
# [0.0, 0.5, 0.0, 0.5]
~~~

A Boolean True means failure; False means success.

### Clarifications to ask

- Are timestamps sorted? Yes, non-decreasing.
- Can multiple events share a timestamp? Yes.
- Does the current event belong to its own window? Yes.
- Is the left boundary included? No.
- What happens when the window contains no events? This cannot occur for the current event because the current event is included.
- Is the result a percentage or fraction? A fraction from 0.0 through 1.0.

### Practice contract

- Input is a list of (timestamp, failed) pairs.
- Timestamps are integers or floats and are non-decreasing.
- failed is a Boolean.
- W is a positive number.
- The window for event time t is (t - W, t].
- Return a list of floating-point failure rates, one per event.
- Empty input returns an empty list.
- Raise ValueError if W is not positive or timestamps are out of order.

### Interviewer guide

**Expected pattern:** Sliding time window using a deque and a running failure count.

For each event:
1. Add the current event.
2. While the oldest event timestamp is less than or equal to t - W, remove it.
3. Compute failures divided by the number of events still in the window.

The order matters: the current event is added, and expired events are removed using the exact left-open boundary.

**Hint 1:** Do you need to recount every event in the window?

**Hint 2:** What small piece of running state can be updated when events enter and leave?

### Reference solution

~~~python
from collections import deque


def rolling_failure_rates(
    events: list[tuple[float, bool]],
    window: float,
) -> list[float]:
    if window <= 0:
        raise ValueError("window must be positive")

    active: deque[tuple[float, bool]] = deque()
    failures = 0
    rates: list[float] = []
    previous_timestamp: float | None = None

    for timestamp, failed in events:
        if previous_timestamp is not None and timestamp < previous_timestamp:
            raise ValueError("timestamps must be non-decreasing")

        previous_timestamp = timestamp
        active.append((timestamp, failed))
        if failed:
            failures += 1

        cutoff = timestamp - window
        while active and active[0][0] <= cutoff:
            _, expired_failed = active.popleft()
            if expired_failed:
                failures -= 1

        rates.append(failures / len(active))

    return rates


assert rolling_failure_rates(
    [(1, False), (2, True), (4, True), (5, False)],
    3,
) == [0.0, 0.5, 0.0, 0.5]

assert rolling_failure_rates([], 5) == []
assert rolling_failure_rates([(1, True), (1, False)], 2) == [1.0, 0.5]
assert rolling_failure_rates([(1, True), (4, False)], 3) == [1.0, 0.0]
~~~

### Boundary trace

For W = 3 and an event at t = 4, the window is (1, 4]. An event at timestamp 1 is excluded because the left boundary is open. Events at timestamps 2 and 4 remain.

### Correctness reasoning

The deque stores exactly the events not yet expired after each iteration. The current event is appended. Every event with timestamp less than or equal to t - W is removed, which enforces the left-open boundary. Because timestamps are non-decreasing, expired events always occur at the front. The failures counter is incremented on insertion and decremented when a failed event expires. Therefore, after pruning, failures divided by deque length is the failure fraction for the specified window.

### Complexity

Let n be the number of events and m the maximum number of events simultaneously retained.

- **Time:** O(n) total amortized. Each event is appended once and removed at most once.
- **Auxiliary space:** O(m) for the active deque, plus O(n) for the returned list.

### Follow-up questions

1. What changes if the left boundary is closed?
2. What if events arrive out of order?
3. How would you calculate separate rates per service?
4. What if the input is an unbounded stream?
5. How would you handle missing telemetry or duplicate event delivery?

### Production-context discussion

This is a deterministic in-memory exercise over already ordered events. A production metric pipeline may need event-time semantics, late-arriving data handling, distributed aggregation, retention policies, deduplication, and explicit treatment of missing observations. Do not present this function as a replacement for a monitoring system.

### Debrief

- Did the candidate define the interval precisely?
- Did they use the correct equality condition at the cutoff?
- Did they update the running failure count on both insertion and expiration?
- Did they explain the assumptions about event ordering?

---

## 8. Mock 5 — Debug a retry scheduler

<badge color="warning">Debugging / Heap</badge>

### Candidate prompt

The following function is intended to return jobs in ascending order of their scheduled retry time. Each input is a pair (job_name, retry_at).

The implementation has a bug. Find it, explain a failing input, fix it, and add tests.

~~~python
def scheduled_jobs(jobs):
    import heapq

    heap = []
    for job_name, retry_at in jobs:
        heapq.heappush(heap, (job_name, retry_at))

    result = []
    while heap:
        job_name, retry_at = heapq.heappop(heap)
        result.append((job_name, retry_at))

    return result
~~~

### Clarifications to ask

- Should jobs be ordered by retry time? Yes, ascending.
- Can two jobs have the same retry time? Yes.
- Is a deterministic tie-breaker required? Yes, job name ascending.
- Are job names strings? Yes.
- Is this function responsible for sleeping or executing jobs? No. It only returns an ordering.

### Interviewer guide

The tuple is pushed as (job_name, retry_at), so Python's tuple ordering prioritizes the job name, not the retry time. The required ordering is (retry_at, job_name).

Ask the candidate to demonstrate the bug with an input where alphabetical order conflicts with retry-time order.

### Failing example

~~~python
jobs = [
    ("zeta", 1),
    ("alpha", 10),
]
~~~

The intended order is zeta at time 1, then alpha at time 10. The broken implementation orders alpha first because the first tuple element is compared first.

### Corrected implementation

~~~python
import heapq


def scheduled_jobs(
    jobs: list[tuple[str, int]],
) -> list[tuple[str, int]]:
    heap: list[tuple[int, str]] = []

    for job_name, retry_at in jobs:
        heapq.heappush(heap, (retry_at, job_name))

    result: list[tuple[str, int]] = []

    while heap:
        retry_at, job_name = heapq.heappop(heap)
        result.append((job_name, retry_at))

    return result


assert scheduled_jobs([
    ("zeta", 1),
    ("alpha", 10),
]) == [("zeta", 1), ("alpha", 10)]

assert scheduled_jobs([
    ("worker-b", 5),
    ("worker-a", 5),
]) == [("worker-a", 5), ("worker-b", 5)]

assert scheduled_jobs([]) == []
~~~

### Correctness reasoning

Each heap entry is stored as (retry_at, job_name). Python's min-heap returns the smallest tuple first, so retry time is the primary ordering key and job name breaks ties. Every input job is inserted once and removed once; therefore the result contains all jobs in the required deterministic order.

### Complexity

For n jobs:

- **Time:** O(n log n) for n heap insertions and n removals.
- **Auxiliary space:** O(n) for the heap and returned list.

If the only requirement is a complete sorted result, sorting the input with an appropriate key is another valid approach. The heap is useful when the exercise is specifically about priority-queue operations or when a system consumes the next item incrementally.

### Follow-up questions

1. What if two jobs can have identical retry time and identical job name?
2. How would you add a stable insertion-order tie-breaker?
3. How would you support cancellation?
4. What additional state is required for exponential backoff?
5. Why should a real retry system distinguish retry scheduling from retry execution?

### Debrief

- Did the candidate create a minimal failing example?
- Did they identify tuple comparison order?
- Did they test ties and empty input?
- Did they avoid adding unrelated production behavior to a focused bug fix?

---

## 9. Mock 6 — Python code review and trade-offs

<badge color="info">Python engineering</badge>

### Candidate prompt

Review this function. It is intended to group service names by their first character, preserving the order in which names appear within each group.

~~~python
def group_services(names, groups={}):
    for name in names:
        key = name[0].lower()
        if key not in groups:
            groups[key] = []
        groups[key].append(name)
    return groups
~~~

Identify correctness and maintainability concerns. Propose a safer implementation and tests.

### Clarifications to ask

- Can names be empty strings? For this exercise, yes; ignore empty names.
- Should grouping be case-insensitive? Yes, based on the first character.
- Should the function preserve input order within groups? Yes.
- Should data from earlier calls remain in the result? No.
- What should happen for non-string input? Raise TypeError.

### Interviewer guide

Expected observations:

1. The mutable default dictionary is created once when the function is defined and reused across calls.
2. Empty strings cause an index error when reading name[0].
3. The implementation does not validate input types.
4. The function mutates a caller-provided dictionary if one is supplied, though the prompt does not require that feature.
5. The return contract should be clear.

Ask the candidate to distinguish a bug from a design choice. For example, whether to ignore empty strings is a contract decision; shared mutable default state is an implementation defect for this stated behavior.

### Reference solution

~~~python
from collections import defaultdict


def group_services(names: list[str]) -> dict[str, list[str]]:
    groups: defaultdict[str, list[str]] = defaultdict(list)

    for name in names:
        if not isinstance(name, str):
            raise TypeError("every service name must be a string")

        if not name:
            continue

        key = name[0].lower()
        groups[key].append(name)

    return dict(groups)


assert group_services(["API", "agent", "Worker", "web"]) == {
    "a": ["API", "agent"],
    "w": ["Worker", "web"],
}

assert group_services(["", "Cache"]) == {"c": ["Cache"]}
assert group_services([]) == {}
assert group_services(["db"]) == {"d": ["db"]}
~~~

### Correctness reasoning

A new local mapping is created for every call, so results cannot leak between invocations. Each non-empty string is assigned to the group identified by its lowercased first character. Appending while traversing the input preserves the relative order of names within each group. Empty strings are skipped according to the practice contract, and non-string values are rejected explicitly.

### Complexity

Let n be the number of names and L the total number of characters across all names.

- **Time:** O(n + L) if accounting for validation and first-character normalization at a high level; each name is visited once. For ordinary string inputs, only the first character is normalized.
- **Auxiliary space:** O(n) for the grouped output, excluding the input.

### Follow-up questions

1. How would you group by the first two characters?
2. How would you make grouping independent of locale-specific case rules?
3. What if the output must be sorted by group key?
4. When is a plain dictionary preferable to defaultdict?
5. How would you test that separate calls do not share state?

### Debrief

- Did the candidate recognize mutable default arguments?
- Did they ask about empty input values?
- Did they separate input validation from grouping behavior?
- Did they write a test that calls the function twice?

---

## 10. Interviewer and candidate instructions

### Running a partner mock

1. Agree on one scenario and the time limit before starting.
2. The interviewer reads only the candidate prompt and practice contract clarifications when asked.
3. The candidate shares their reasoning aloud and writes code in a shared editor or on paper.
4. The interviewer records observations and hints without coaching.
5. Reserve time for tests, complexity, and follow-ups.
6. Stop the coding portion at the agreed time, even if incomplete.
7. Complete the scorecard independently before discussing the result.
8. End with two strengths, two improvement areas, and one action for the next session.

### Role rotation

For two people, alternate candidate and interviewer roles. The interviewer should solve the problem independently before using it in a session, so they can recognize valid alternative approaches and avoid treating the reference solution as the only correct answer.

For a group, one person can interview, one can observe communication, and one can review tests and complexity. Rotate roles in later sessions.

### Interviewer hint ladder

Use the smallest hint that helps the candidate make progress:

1. **Clarification prompt:** “What assumptions do you need to confirm?”
2. **Model prompt:** “Can you walk through a small example?”
3. **Data-structure prompt:** “What operation needs to be efficient?”
4. **Pattern prompt:** “Would a sliding window or graph traversal fit?”
5. **Implementation prompt:** Point to a specific state update or boundary.

Record the hint level. A solution reached with a strong hint is useful learning, but it is different evidence from an independently derived solution.

## 11. Evaluation scorecard

Score each dimension from 0 to 4. Use the descriptions as feedback anchors, not as a universal hiring rubric.

| Dimension | 0 — Not demonstrated | 2 — Developing | 4 — Strong |
|---|---|---|---|
| Problem understanding | Misstates the task | Understands the main goal; misses a constraint | Clearly defines input, output, and boundaries |
| Clarification | Makes unsupported assumptions | Asks some useful questions | Resolves important ambiguities efficiently |
| Approach | No workable approach | Direction is plausible but incomplete | Explains a suitable approach and trade-offs |
| Correctness | Code does not address the task | Main path works; important cases fail | Reasoning and implementation cover the contract |
| Code quality | Difficult to follow | Understandable with avoidable issues | Clear names, focused logic, appropriate structure |
| Testing | No meaningful tests | Tests a normal case | Tests normal, boundary, and edge cases |
| Complexity | No analysis | Gives incomplete or imprecise analysis | Defines variables and justifies time/space costs |
| Communication | Reasoning is not explained | Explains some decisions | Communicates clearly and responds constructively |

**Maximum practice score: 32 points.**

Suggested interpretation for personal tracking only:

- **0–15:** Revisit the relevant concept and solve the problem untimed.
- **16–23:** Core understanding is emerging; focus on edge cases and explanation.
- **24–28:** Good practice result; repeat with a different problem using the same pattern.
- **29–32:** Strong performance on this exercise; test transfer with an unfamiliar variation.

These bands are repository-created coaching guidance. They are not validated thresholds and should not be compared with employer hiring scores.

### Record hints separately

| Hint level | Meaning |
|---|---|
| 0 | No hint needed |
| 1 | Clarification or example prompt |
| 2 | Data-structure or modeling prompt |
| 3 | Pattern-level hint |
| 4 | Specific implementation guidance |

Do not subtract points mechanically based on hints. Record them to identify what to practise next.

## 12. Follow-up question bank

Use these after the candidate has a working solution.

### Correctness and edge cases

- What happens with empty input?
- What if every value is identical?
- What if there are duplicate relationships or repeated events?
- Which boundary condition is easiest to get wrong?
- Can you explain why the loop invariant remains true?

### Complexity and scale

- What are the time and auxiliary-space costs?
- Which operation dominates the runtime?
- What changes if the input is ten times larger?
- Is the complexity worst-case or expected?
- Can the output itself require linear memory?

### Design alternatives

- What is the simplest correct solution?
- Which alternative data structure could work?
- What trade-off does the alternative introduce?
- Would sorting simplify the implementation?
- When would a heap be more useful than sorting?

### Python fluency

- Which standard-library container is suitable here?
- What assumptions are made about hashability?
- Does this function mutate its input?
- Could a mutable default argument cause state leakage?
- How would you write a focused unit test?

### DevOps and SRE context

- What happens if events arrive late or out of order?
- How would you handle duplicate messages?
- What state must survive a process restart?
- What changes when multiple workers process the same queue?
- Which failure modes are outside this algorithm's scope?
- What metrics would help an operator understand the behavior?

## 13. Debrief and improvement plan

Complete this immediately after each session.

~~~text
Mock number and title:
Date:
Practice mode: solo / partner / group
Time spent:
Completed independently? yes / partly / no
Hint level used: 0 / 1 / 2 / 3 / 4

What I did well:
1.
2.

Where I lost time:
1.
2.

Correctness issue(s):
1.

Missed test case(s):
1.

Complexity explanation:
What I would change in the solution:

Score:
Problem pattern to revise:
One specific action before the next mock:
~~~

### Turn feedback into practice

Avoid vague actions such as “study graphs more.” Prefer a measurable next step:

- “Implement topological sort twice: once with a queue and once with a min-heap.”
- “Write five tests for open and closed interval boundaries.”
- “Explain the sliding-window invariant without looking at notes.”
- “Solve one new hash-map problem and state the expected-time assumption.”
- “Run the same mock again in a week without reading the reference solution.”

A repeated mock is useful for measuring improvement, but also solve an unseen problem from the same pattern to check whether the learning transfers.

## 14. Solo and remote practice

### Solo mode

1. Set a timer for the suggested duration.
2. Read only the candidate prompt.
3. Speak your clarification questions aloud and write down the answers you assume.
4. Record a short explanation of your approach before coding.
5. Implement the solution without looking at the reference.
6. Run tests and explain complexity aloud.
7. Review the interviewer guide and score yourself conservatively.
8. Write one action for the next session.

If you cannot run code during the mock, trace it manually and mark execution as unverified. Do not treat a mental trace as a passing test run.

### Remote partner mode

- Share the prompt and code editor, but keep the interviewer guide private.
- Agree on whether documentation and execution are allowed before starting.
- Ask the candidate to narrate important decisions rather than every keystroke.
- Keep feedback until the timed portion ends.
- Share the scorecard and debrief notes afterward.

### Reproducible testing

The Python standard library includes unittest for organizing and running automated tests. Use it when turning a mock solution into a small practice module. See the official [unittest documentation](https://docs.python.org/3/library/unittest.html).

For small timing experiments, Python provides timeit. Benchmark only after correctness is established, and avoid treating one machine's timing as a universal performance result. See [timeit documentation](https://docs.python.org/3/library/timeit.html).

For debugging, pdb provides an interactive source-level debugger with breakpoints and stepping. See [pdb documentation](https://docs.python.org/3/library/pdb.html).

## 15. Readiness checklist

You are building useful interview habits when you can consistently:

- [ ] Restate a problem and identify its input/output contract.
- [ ] Ask about constraints that affect the solution.
- [ ] Recognize common patterns without relying on memorized problem titles.
- [ ] Explain an approach before coding.
- [ ] Implement readable Python with appropriate built-in data structures.
- [ ] State and preserve a useful invariant.
- [ ] Test ordinary, empty, duplicate, and boundary cases where relevant.
- [ ] Explain time and auxiliary-space complexity accurately.
- [ ] Debug a failing example systematically.
- [ ] Discuss a simpler alternative and its trade-offs.
- [ ] Identify when an algorithmic exercise omits production concerns.
- [ ] Receive feedback and turn it into a specific practice action.

Do not use one score or one successful mock as proof of universal readiness. Repeat across patterns and solve unfamiliar variations.

## 16. References and navigation

### Official Python documentation

- [unittest — Unit testing framework](https://docs.python.org/3/library/unittest.html) — test cases, fixtures, suites, and runners.
- [collections — Container datatypes](https://docs.python.org/3/library/collections.html) — Counter, deque, defaultdict, and other specialized containers.
- [heapq — Heap queue algorithm](https://docs.python.org/3/library/heapq.html) — min-heap operations used in priority-queue exercises.
- [pdb — The Python debugger](https://docs.python.org/3/library/pdb.html) — breakpoints, stepping, and inspection.
- [logging — Logging facility for Python](https://docs.python.org/3/library/logging.html) — application event logging.
- [timeit — Measure execution time of small code snippets](https://docs.python.org/3/library/timeit.html) — controlled timing of small code snippets.

The reference solutions and interview scenarios are repository-authored educational material. The linked documentation supports Python API behavior; it does not certify the exercise contracts, scorecard, or suggested interview timings.

### Continue the interview-preparation path

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
- [11 — Revision and Progress Tracking](../11%20Revision%20and%20Progress%20Tracking/README.md) — planned next section.
