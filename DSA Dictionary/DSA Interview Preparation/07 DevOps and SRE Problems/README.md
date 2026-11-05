# 07 — DevOps and SRE Problems

This section applies data structures and algorithms to operationally familiar problems: log analysis, alert processing, rate limiting, retry scheduling, dependency analysis, capacity planning, and reliability measurement.

> **Learning goal:** Translate an operational scenario into a precise data contract, choose an appropriate algorithm, implement it in Python, and explain its limitations before treating it as a production design.

These are **educational simulations**, not production replacements for Kubernetes, Prometheus, an API gateway, a message broker, or an incident-management platform. Official platform documentation grounds the scenario context. Input formats, simplified policies, and algorithms are repository design decisions.

---

## How to approach an operational problem

Operational language is often ambiguous. Before writing code:

1. Define the event or resource and its fields.
2. Define time semantics: ordering, window boundaries, and late events.
3. Define the policy: what counts as a failure, retry, duplicate, or unavailable service?
4. Define the required output.
5. Choose a suitable structure: counter, deque, heap, graph, disjoint set, or sweep line.
6. State the invariant maintained while processing.
7. Test empty input, repeated events, exact time boundaries, missing dependencies, cycles, and invalid records.
8. State what the model omits. Production systems add concurrency, persistence, distributed coordination, observability, security, and failure handling.

## Problem map

| # | Problem | Main technique | Operational connection |
|---:|---|---|---|
| 1 | Find the most common failure signatures | Counter + heap | Log and event triage |
| 2 | Calculate a rolling failure rate | Sliding window + deque | Time-window monitoring |
| 3 | Simulate a token-bucket rate limiter | Time-based state | API traffic shaping |
| 4 | Schedule retry attempts | Min-heap | Transient job failures |
| 5 | Find services affected by a dependency failure | Reverse graph + BFS | Incident blast-radius analysis |
| 6 | Calculate the critical path in a dependency DAG | Topological sort + DP | Delivery and recovery planning |
| 7 | Find peak concurrent resource usage | Sweep line | Deployment and capacity overlap |
| 8 | Calculate an SLO error budget | Reliability arithmetic | SLO tracking |
| 9 | Suppress repeated alerts during a cooldown | Hash map | Alert deduplication |
| 10 | Group incidents connected by shared resources | Disjoint Set Union | Incident correlation |

---

## 1. Find the most common failure signatures

**Pattern:** Frequency map + heap

### Problem

Given log-event records, return the K most frequent failure signatures. A signature is the pair (service, error_code). Only records whose status is exactly "error" are counted.

**Contract and assumptions**

- Each record is a mapping with service, status, and error_code string fields.
- K must be a non-negative integer.
- Return at most K signatures.
- Sort by descending frequency, then ascending service name, then ascending error code.
- This groups exact string values. It does not parse free-form log messages or normalize error codes.

**Example**

~~~python
events = [
    {"service": "payments", "status": "error", "error_code": "TIMEOUT"},
    {"service": "catalog", "status": "ok", "error_code": ""},
    {"service": "payments", "status": "error", "error_code": "TIMEOUT"},
    {"service": "catalog", "status": "error", "error_code": "DB_DOWN"},
]
assert top_failure_signatures(events, 2) == [
    (("payments", "TIMEOUT"), 2),
    (("catalog", "DB_DOWN"), 1),
]
~~~

### Hint

Count signatures in one pass. Select the K smallest ranking keys after negating the count. For a small or moderate number of distinct signatures, sorting may be clearer; a heap is useful when K is much smaller than the number of unique signatures.

### Worked solution

~~~python
from collections import Counter
import heapq


def top_failure_signatures(events, k):
    if k < 0:
        raise ValueError("k cannot be negative")

    counts = Counter()
    for event in events:
        if event["status"] == "error":
            signature = (event["service"], event["error_code"])
            counts[signature] += 1

    ranked = [
        (-count, service, error_code)
        for (service, error_code), count in counts.items()
    ]
    selected = heapq.nsmallest(k, ranked)

    return [
        ((service, error_code), -negative_count)
        for negative_count, service, error_code in selected
    ]


sample = [
    {"service": "payments", "status": "error", "error_code": "TIMEOUT"},
    {"service": "catalog", "status": "ok", "error_code": ""},
    {"service": "payments", "status": "error", "error_code": "TIMEOUT"},
    {"service": "catalog", "status": "error", "error_code": "DB_DOWN"},
]
assert top_failure_signatures(sample, 2) == [
    (("payments", "TIMEOUT"), 2),
    (("catalog", "DB_DOWN"), 1),
]
assert top_failure_signatures([], 5) == []
assert top_failure_signatures(sample, 0) == []
assert top_failure_signatures(sample, 10)[-1] == (("catalog", "DB_DOWN"), 1)
~~~

### Why it works

The counter stores occurrences of each signature. The selection key is (negative count, service, error code), so smaller keys correspond to higher counts and deterministic alphabetical tie-breaking.

Let N be the number of events, U the number of distinct failure signatures, and K the requested result count:

- **Counting time:** O(N) expected, assuming average constant-time dictionary operations.
- **Selection time:** O(U log K) when K is smaller than U; the library may use a different strategy when K is large.
- **Space:** O(U).

**Common mistake:** Grouping only by error code when the same code can have different meanings across services.

**Operational limitation:** Prometheus labels identify dimensions of time series. A change in label value creates a different series. Do not assume every arbitrary log field should become a metric label; uncontrolled label cardinality can make a metrics design expensive. This is in-memory event aggregation, not a Prometheus storage model.

---

## 2. Calculate a rolling failure rate

**Pattern:** Sliding window + deque

### Problem

Given request events sorted by timestamp, calculate the failure rate for the trailing time window at each event. Each event is (timestamp, failed), where failed is Boolean.

For an event at time T and window width W, include timestamps satisfying:

**T - W < timestamp <= T**

Return (timestamp, failure_rate) for every input event.

**Contract and assumptions**

- Timestamps are non-decreasing integers in seconds.
- W is a positive integer.
- Every event is one request and contributes one to the denominator.
- The left boundary is exclusive and the right boundary is inclusive.
- Equal timestamps are included as events are processed in input order.

**Example**

~~~python
requests = [(1, False), (2, True), (3, False), (7, True)]
assert rolling_failure_rates(requests, 3) == [
    (1, 0.0), (2, 0.5), (3, 1 / 3), (7, 1.0)
]
~~~

### Hint

Keep active events in a deque. Remove entries at or before the cutoff and maintain a running failure count.

### Worked solution

~~~python
from collections import deque


def rolling_failure_rates(events, window_seconds):
    if window_seconds <= 0:
        raise ValueError("window_seconds must be positive")

    active = deque()
    failures = 0
    previous_time = None
    results = []

    for timestamp, failed in events:
        if previous_time is not None and timestamp < previous_time:
            raise ValueError("events must be sorted by timestamp")
        if not isinstance(failed, bool):
            raise TypeError("failed must be a Boolean")

        previous_time = timestamp
        active.append((timestamp, failed))
        if failed:
            failures += 1

        cutoff = timestamp - window_seconds
        while active and active[0][0] <= cutoff:
            _, expired_failed = active.popleft()
            if expired_failed:
                failures -= 1

        results.append((timestamp, failures / len(active)))

    return results


assert rolling_failure_rates(
    [(1, False), (2, True), (3, False), (7, True)], 3
) == [(1, 0.0), (2, 0.5), (3, 1 / 3), (7, 1.0)]
assert rolling_failure_rates([], 5) == []
assert rolling_failure_rates([(5, True), (5, False)], 1) == [
    (5, 1.0), (5, 0.5)
]
~~~

### Why it works

After expired events are removed, the deque contains exactly the events in the defined trailing window. The running failure count is updated when a failed event enters or leaves. Each event is appended once and removed at most once.

- **Time:** O(N) total.
- **Auxiliary space:** O(M), where M is the maximum number of events in one active window.

**Common mistake:** Including the left boundary when the contract says it is exclusive.

**Operational limitation:** This model assumes events arrive in timestamp order. Real telemetry pipelines may receive delayed or out-of-order samples and need a lateness policy, buffering strategy, and possibly recomputation. Prometheus supports instant and range queries over time series; this Python exercise is a smaller event-stream model.

---

## 3. Simulate a token-bucket rate limiter

**Pattern:** Time-based state machine

### Problem

Implement a token bucket that decides whether a request is allowed at a supplied timestamp. The bucket has a maximum capacity and refills at a fixed number of tokens per second. Each accepted request consumes one token.

**Contract and assumptions**

- Capacity is positive.
- Refill rate is non-negative tokens per second.
- The bucket starts full.
- Timestamps are numeric and non-decreasing.
- A request is accepted only when at least one token is available.
- Fractional tokens are allowed.
- This is a single-process deterministic simulation, not a distributed rate limiter.

**Example**

~~~python
bucket = TokenBucket(capacity=2, refill_rate=1, start_time=0)
assert bucket.allow(0) is True
assert bucket.allow(0) is True
assert bucket.allow(0) is False
assert bucket.allow(1) is True
~~~

### Hint

Store the current token count and last observed timestamp. Before each decision, add elapsed time multiplied by the refill rate, capped at capacity.

### Worked solution

~~~python
class TokenBucket:
    def __init__(self, capacity, refill_rate, start_time=0):
        if capacity <= 0:
            raise ValueError("capacity must be positive")
        if refill_rate < 0:
            raise ValueError("refill_rate cannot be negative")

        self.capacity = float(capacity)
        self.refill_rate = float(refill_rate)
        self.tokens = float(capacity)
        self.last_time = float(start_time)

    def allow(self, timestamp):
        timestamp = float(timestamp)
        if timestamp < self.last_time:
            raise ValueError("timestamps must be non-decreasing")

        elapsed = timestamp - self.last_time
        self.tokens = min(
            self.capacity,
            self.tokens + elapsed * self.refill_rate
        )
        self.last_time = timestamp

        if self.tokens < 1.0:
            return False

        self.tokens -= 1.0
        return True


bucket = TokenBucket(2, 1, start_time=0)
assert bucket.allow(0) is True
assert bucket.allow(0) is True
assert bucket.allow(0) is False
assert bucket.allow(1) is True
assert bucket.allow(1.5) is False
assert bucket.allow(2) is True

empty_refill = TokenBucket(1, 0, start_time=0)
assert empty_refill.allow(0) is True
assert empty_refill.allow(100) is False
~~~

### Why it works

Before a request is evaluated, the bucket is advanced to the request timestamp. The token count increases by elapsed time times refill rate, but never exceeds capacity. The request consumes one token only when the count is at least one.

- **Time:** O(1) per request.
- **Auxiliary space:** O(1).

**Common mistake:** Allowing the token count to grow above capacity after a long idle period.

**Production limitation:** Distributed rate limiting needs shared or partitioned state, concurrency control, clock considerations, and defined behavior when the state store is unavailable. Floating-point precision and process restarts also matter.

---

## 4. Schedule retry attempts

**Pattern:** Min-heap priority queue

### Problem

Given jobs represented as (job_id, first_failure_time, max_attempts), produce their retry schedule using exponential backoff. The first retry is scheduled after base_delay; each later retry delay doubles.

**Contract and assumptions**

- Job identifiers are unique strings.
- Times and delays are non-negative integers.
- max_attempts counts the initial attempt plus retries. A value of 1 means no retry.
- A retry is emitted only when its scheduled time is no later than horizon.
- Simultaneous retries are ordered by job ID, then attempt number.
- This calculates a schedule; it does not execute jobs or decide whether a failure is transient.

**Example**

~~~python
jobs = [("worker-a", 10, 3), ("worker-b", 10, 2)]
assert retry_schedule(jobs, base_delay=2, horizon=20) == [
    (12, "worker-a", 2),
    (12, "worker-b", 2),
    (16, "worker-a", 3),
]
~~~

### Hint

Push the next retry into a min-heap as (scheduled_time, job_id, attempt). Pop the earliest retry, record it, then schedule the following attempt if one remains and it falls within the horizon.

### Worked solution

~~~python
import heapq


def retry_schedule(jobs, base_delay, horizon):
    if base_delay < 0 or horizon < 0:
        raise ValueError("delay and horizon must be non-negative")

    pending = []
    seen_ids = set()

    for job_id, first_failure_time, max_attempts in jobs:
        if job_id in seen_ids:
            raise ValueError("job identifiers must be unique")
        seen_ids.add(job_id)

        if first_failure_time < 0:
            raise ValueError("failure time must be non-negative")
        if max_attempts < 1:
            raise ValueError("max_attempts must be at least one")

        if max_attempts > 1:
            first_retry = first_failure_time + base_delay
            if first_retry <= horizon:
                heapq.heappush(
                    pending, (first_retry, job_id, 2, max_attempts)
                )

    schedule = []
    while pending:
        retry_time, job_id, attempt, max_attempts = heapq.heappop(pending)
        schedule.append((retry_time, job_id, attempt))

        if attempt < max_attempts:
            next_delay = base_delay * (2 ** (attempt - 1))
            next_time = retry_time + next_delay
            if next_time <= horizon:
                heapq.heappush(
                    pending,
                    (next_time, job_id, attempt + 1, max_attempts)
                )

    return schedule


assert retry_schedule(
    [("worker-a", 10, 3), ("worker-b", 10, 2)], 2, 20
) == [
    (12, "worker-a", 2),
    (12, "worker-b", 2),
    (16, "worker-a", 3),
]
assert retry_schedule([("one-shot", 0, 1)], 3, 10) == []
assert retry_schedule([("late", 10, 3)], 2, 11) == []
assert retry_schedule([], 1, 10) == []
~~~

### Why it works

The heap always exposes the earliest scheduled retry. When an entry is removed, its next attempt—if permitted—is calculated and inserted. This maintains chronological output without repeatedly sorting the entire schedule.

Let J be the number of jobs and R the number of retries emitted:

- **Time:** O((J + R) log(J + R)) as a broad bound.
- **Auxiliary space:** O(J + R) including the pending heap and output.

**Common mistake:** Doubling the wrong delay. Attempt 2 uses base_delay; attempt 3 uses twice base_delay; attempt 4 uses four times base_delay.

**Production limitation:** Real retry policies may add jitter, maximum delay, retry classification, cancellation, idempotency, dead-letter handling, and a global retry budget.

---

## 5. Find services affected by a dependency failure

**Pattern:** Reverse graph + breadth-first search

### Problem

Given directed dependency edges (service, dependency), return every service that depends directly or indirectly on a failed service.

If checkout depends on payments, and web depends on checkout, a failure in payments affects checkout and web.

**Contract and assumptions**

- An edge (A, B) means A requires B.
- The graph may contain cycles.
- The failed service may not appear in any edge.
- The failed service itself is excluded from the result.
- Return affected services alphabetically.

**Example**

~~~python
dependencies = [
    ("web", "checkout"),
    ("checkout", "payments"),
    ("checkout", "inventory"),
    ("reports", "warehouse"),
]
assert affected_services(dependencies, "payments") == ["checkout", "web"]
~~~

### Hint

Build the reverse adjacency list: for each dependency B, record services that require B. Traverse outward from the failed service.

### Worked solution

~~~python
from collections import defaultdict, deque


def affected_services(dependencies, failed_service):
    dependents = defaultdict(set)
    for service, dependency in dependencies:
        dependents[dependency].add(service)

    affected = set()
    visited = {failed_service}
    pending = deque([failed_service])

    while pending:
        unavailable = pending.popleft()
        for service in dependents[unavailable]:
            if service in visited:
                continue
            visited.add(service)
            affected.add(service)
            pending.append(service)

    return sorted(affected)


dependencies = [
    ("web", "checkout"),
    ("checkout", "payments"),
    ("checkout", "inventory"),
    ("reports", "warehouse"),
]
assert affected_services(dependencies, "payments") == ["checkout", "web"]
assert affected_services(dependencies, "unknown") == []
assert affected_services([("a", "b"), ("b", "a")], "a") == ["b"]
assert affected_services([("a", "b"), ("a", "b")], "b") == ["a"]
~~~

### Why it works

An edge A → B in the original representation means A requires B. Reversing the edge makes B point to its dependents. Breadth-first search from the failed dependency visits every service reachable through one or more dependency relationships. The visited set prevents repeated processing and terminates even when cycles exist.

Let V be the number of services and E the number of dependency edges:

- **Time:** O(V + E) for traversal, plus O(A log A) to sort A affected services.
- **Auxiliary space:** O(V + E).

**Common mistake:** Traversing the original edge direction. That finds what a service depends on, not what depends on the failed service.

**Operational limitation:** A dependency graph is only as accurate as its source data. Optional dependencies, degraded-but-available services, caching, traffic routing, and failure isolation are not represented by this binary model.

---

## 6. Calculate the critical path in a dependency DAG

**Pattern:** Topological sort + dynamic programming

### Problem

Given tasks with durations and prerequisite edges (task, prerequisite), calculate the earliest finish time for each task and the total completion time when tasks can run in parallel as soon as all prerequisites are complete.

**Contract and assumptions**

- Task names are unique strings.
- Durations are non-negative integers.
- Every prerequisite edge references a known task.
- The dependency graph must be acyclic.
- A task with no prerequisites can start at time zero.
- Unlimited parallel execution is assumed.

**Example**

~~~python
durations = {"build": 3, "test": 2, "scan": 4, "deploy": 1}
prerequisites = [
    ("test", "build"),
    ("scan", "build"),
    ("deploy", "test"),
    ("deploy", "scan"),
]
finish, total = critical_path(durations, prerequisites)
assert finish == {"build": 3, "test": 5, "scan": 7, "deploy": 8}
assert total == 8
~~~

### Hint

Topologically process tasks. The earliest start of a task is the maximum earliest finish among its prerequisites. Add the task duration to obtain its earliest finish.

### Worked solution

~~~python
from collections import defaultdict, deque


def critical_path(durations, prerequisites):
    if any(duration < 0 for duration in durations.values()):
        raise ValueError("task durations cannot be negative")

    dependents = defaultdict(list)
    parents = {task: [] for task in durations}
    indegree = {task: 0 for task in durations}

    for task, prerequisite in prerequisites:
        if task not in durations or prerequisite not in durations:
            raise ValueError("every dependency must reference a known task")
        dependents[prerequisite].append(task)
        parents[task].append(prerequisite)
        indegree[task] += 1

    pending = deque(
        task for task, degree in indegree.items() if degree == 0
    )
    earliest_finish = {}
    processed = 0

    while pending:
        task = pending.popleft()
        processed += 1
        earliest_start = max(
            (earliest_finish[parent] for parent in parents[task]),
            default=0
        )
        earliest_finish[task] = earliest_start + durations[task]

        for dependent in dependents[task]:
            indegree[dependent] -= 1
            if indegree[dependent] == 0:
                pending.append(dependent)

    if processed != len(durations):
        raise ValueError("dependency graph contains a cycle")

    return earliest_finish, max(earliest_finish.values(), default=0)


durations = {"build": 3, "test": 2, "scan": 4, "deploy": 1}
prerequisites = [
    ("test", "build"),
    ("scan", "build"),
    ("deploy", "test"),
    ("deploy", "scan"),
]
finish, total = critical_path(durations, prerequisites)
assert finish == {"build": 3, "test": 5, "scan": 7, "deploy": 8}
assert total == 8
assert critical_path({}, []) == ({}, 0)

try:
    critical_path({"a": 1, "b": 1}, [("a", "b"), ("b", "a")])
    raise AssertionError("cycle should be rejected")
except ValueError:
    pass
~~~

### Why it works

A topological order ensures all prerequisites of a task are processed before the task itself. The earliest possible start is the maximum completion time among its prerequisites. Adding the task duration gives its earliest finish. The maximum finish across all tasks is the project completion time under the unlimited-parallelism assumption.

- **Time:** O(V + E).
- **Auxiliary space:** O(V + E).

**Important:** This is a dependency-graph calculation, not a resource-constrained scheduler. If tasks compete for a fixed number of runners or machines, the problem changes.

If fewer than V tasks are processed by Kahn's algorithm, a directed cycle exists and no complete prerequisite ordering is possible.

---

## 7. Find peak concurrent resource usage

**Pattern:** Sweep line

### Problem

Given resource reservations represented as (start, end, units), return the maximum number of units reserved at the same time.

**Contract and assumptions**

- Times and units are integers.
- Every interval is half-open: [start, end).
- Each interval must have start < end and units > 0.
- Reservations ending at T do not overlap reservations beginning at T.
- Empty input returns zero.

**Example**

~~~python
reservations = [(1, 5, 2), (3, 7, 3), (5, 8, 1)]
assert peak_reserved_units(reservations) == 5
~~~

### Hint

Create a positive change at each start and a negative change at each end. Aggregate all changes at a timestamp before measuring the active total.

### Worked solution

~~~python
from collections import defaultdict


def peak_reserved_units(reservations):
    changes = defaultdict(int)

    for start, end, units in reservations:
        if start >= end:
            raise ValueError("reservation must have positive duration")
        if units <= 0:
            raise ValueError("units must be positive")
        changes[start] += units
        changes[end] -= units

    current = 0
    peak = 0
    for timestamp in sorted(changes):
        current += changes[timestamp]
        peak = max(peak, current)

    return peak


assert peak_reserved_units([(1, 5, 2), (3, 7, 3), (5, 8, 1)]) == 5
assert peak_reserved_units([]) == 0
assert peak_reserved_units([(1, 3, 4), (3, 5, 2)]) == 4
assert peak_reserved_units([(1, 4, 2), (2, 3, 3)]) == 5
~~~

### Why it works

At each timestamp, the aggregated change is the net difference between reservations active immediately before and immediately after that timestamp. Applying all changes together respects half-open intervals: reservations ending at T are no longer active in the span after T, while reservations starting at T are active.

- **Time:** O(N log N) for N reservations due to sorting distinct timestamps.
- **Auxiliary space:** O(N).

**Common mistake:** Treating intervals as closed at both ends. That incorrectly counts reservations that merely touch at a boundary as overlapping.

**Operational application:** Analyze overlapping maintenance windows, deployment reservations, or capacity commitments. This is an offline calculation, not a live cluster scheduler.

---

## 8. Calculate an SLO error budget

**Pattern:** Reliability arithmetic + input validation

### Problem

Given a request count, failed request count, and target availability expressed in basis points, calculate the allowed failure count and remaining error budget. A target of 99.9% availability is 9990 basis points out of 10000.

**Contract and assumptions**

- Request and failure counts are non-negative integers.
- Failed requests cannot exceed total requests.
- Target availability is an integer from 0 through 10000 basis points.
- Allowed failures are rounded down to a whole request.
- Remaining budget may be negative when observed failures exceed the allowed budget.
- Zero requests produce an observed failure rate of None.

**Example**

~~~python
result = error_budget(100_000, 50, 9990)
assert result == {
    "allowed_failures": 100,
    "remaining_failures": 50,
    "observed_failure_rate": 0.0005,
    "budget_consumed": 0.5,
}
~~~

### Hint

The allowed failure fraction is (10000 - target) / 10000. Use integer arithmetic to avoid rounding the allowed count upward.

### Worked solution

~~~python
def error_budget(total_requests, failed_requests, target_availability_bps):
    if total_requests < 0 or failed_requests < 0:
        raise ValueError("request counts cannot be negative")
    if failed_requests > total_requests:
        raise ValueError("failures cannot exceed total requests")
    if not 0 <= target_availability_bps <= 10000:
        raise ValueError("target must be between 0 and 10000 basis points")

    allowed_failure_bps = 10000 - target_availability_bps
    allowed_failures = (
        total_requests * allowed_failure_bps
    ) // 10000
    remaining = allowed_failures - failed_requests

    observed_rate = (
        failed_requests / total_requests if total_requests else None
    )

    if allowed_failures == 0:
        budget_consumed = 0.0 if failed_requests == 0 else float("inf")
    else:
        budget_consumed = failed_requests / allowed_failures

    return {
        "allowed_failures": allowed_failures,
        "remaining_failures": remaining,
        "observed_failure_rate": observed_rate,
        "budget_consumed": budget_consumed,
    }


assert error_budget(100_000, 50, 9990) == {
    "allowed_failures": 100,
    "remaining_failures": 50,
    "observed_failure_rate": 0.0005,
    "budget_consumed": 0.5,
}
assert error_budget(100, 2, 9900)["remaining_failures"] == -1
assert error_budget(0, 0, 10000) == {
    "allowed_failures": 0,
    "remaining_failures": 0,
    "observed_failure_rate": None,
    "budget_consumed": 0.0,
}
assert error_budget(100, 1, 10000)["budget_consumed"] == float("inf")
~~~

### Why it works

The allowed failure fraction is the complement of the availability target. Multiplying by total requests gives the theoretical allowed failures; integer floor ensures the count does not exceed the fractional allowance. Subtracting observed failures yields the remaining budget.

- **Time:** O(1).
- **Auxiliary space:** O(1).

**Important assumptions:** This exercise uses a request-based objective over one supplied population. Real SLOs must define the eligible event population, measurement window, good-event criteria, treatment of missing data, and aggregation method. Different SLIs and SLO policies can produce different calculations. This simple formula is not a universal SLO implementation.

---

## 9. Suppress repeated alerts during a cooldown

**Pattern:** Hash map + time comparison

### Problem

Given alert events (timestamp, fingerprint), emit an alert only if no alert with the same fingerprint has been emitted during the preceding cooldown interval.

**Contract and assumptions**

- Timestamps are non-decreasing integers.
- Fingerprints are strings.
- Cooldown is a non-negative integer.
- An event is suppressed when timestamp - last_emitted < cooldown.
- An event exactly at the cooldown boundary is emitted.
- Suppressed events do not reset the cooldown.
- Preserve input order.

**Example**

~~~python
alerts = [
    (1, "api-down"), (3, "api-down"),
    (6, "api-down"), (7, "db-down")
]
assert suppress_alerts(alerts, 5) == [
    (1, "api-down"), (6, "api-down"), (7, "db-down")
]
~~~

### Hint

Track the timestamp of the last emitted event for each fingerprint. Suppressed events must not reset the cooldown in this policy.

### Worked solution

~~~python
def suppress_alerts(alerts, cooldown):
    if cooldown < 0:
        raise ValueError("cooldown cannot be negative")

    last_emitted = {}
    emitted = []
    previous_time = None

    for timestamp, fingerprint in alerts:
        if previous_time is not None and timestamp < previous_time:
            raise ValueError("alerts must be sorted by timestamp")
        previous_time = timestamp

        last_time = last_emitted.get(fingerprint)
        if last_time is None or timestamp - last_time >= cooldown:
            emitted.append((timestamp, fingerprint))
            last_emitted[fingerprint] = timestamp

    return emitted


alerts = [
    (1, "api-down"), (3, "api-down"),
    (6, "api-down"), (7, "db-down")
]
assert suppress_alerts(alerts, 5) == [
    (1, "api-down"), (6, "api-down"), (7, "db-down")
]
assert suppress_alerts([], 5) == []
assert suppress_alerts([(2, "same"), (2, "same")], 0) == [
    (2, "same"), (2, "same")
]
assert suppress_alerts([(1, "a"), (5, "a")], 4) == [
    (1, "a"), (5, "a")
]
~~~

### Why it works

The map stores the last emitted time for each fingerprint. An incoming event is emitted if the fingerprint has never been emitted or the cooldown has elapsed. Suppressed events do not update the map, matching the stated policy.

- **Time:** O(N) expected for N alert events.
- **Auxiliary space:** O(F), where F is the number of distinct fingerprints.

**Common mistake:** Updating the timestamp for a suppressed alert. That changes the policy into a sliding suppression interval that can postpone emission indefinitely during a continuous stream.

**Operational limitation:** This is a single-process, in-memory rule. Distributed alert processors need coordination, persistence, deduplication semantics, and defined restart behavior.

---

## 10. Group incidents connected by shared resources

**Pattern:** Disjoint Set Union

### Problem

Each incident record contains a set of resource identifiers, such as hostnames, service names, or cluster identifiers. Group incidents into the same component when they share at least one resource directly or through a chain of other incidents.

**Contract and assumptions**

- Incident IDs are unique strings.
- Resource identifiers are strings.
- An incident with no resources forms a group by itself.
- Two incidents are connected if they share a resource or are linked through other incidents.
- Return IDs sorted inside each group, and sort groups by their first ID.
- This is offline batch analysis; it does not infer causality.

**Example**

~~~python
incidents = [
    ("INC-1", ["node-a", "service-payments"]),
    ("INC-2", ["service-payments"]),
    ("INC-3", ["node-z"]),
    ("INC-4", ["node-a"]),
    ("INC-5", []),
]
assert correlate_incidents(incidents) == [
    ["INC-1", "INC-2", "INC-4"],
    ["INC-3"],
    ["INC-5"],
]
~~~

### Hint

Use a disjoint-set structure over incident indices. Keep a map from resource ID to the first incident that referenced it. Union later incidents that reference the same resource.

### Worked solution

~~~python
from collections import defaultdict


class DisjointSet:
    def __init__(self, size):
        self.parent = list(range(size))
        self.size = [1] * size

    def find(self, item):
        if self.parent[item] != item:
            self.parent[item] = self.find(self.parent[item])
        return self.parent[item]

    def union(self, first, second):
        root_first = self.find(first)
        root_second = self.find(second)
        if root_first == root_second:
            return

        if self.size[root_first] < self.size[root_second]:
            root_first, root_second = root_second, root_first

        self.parent[root_second] = root_first
        self.size[root_first] += self.size[root_second]


def correlate_incidents(incidents):
    incident_ids = [incident_id for incident_id, _ in incidents]
    if len(set(incident_ids)) != len(incident_ids):
        raise ValueError("incident IDs must be unique")

    dsu = DisjointSet(len(incidents))
    first_incident_by_resource = {}

    for index, (incident_id, resources) in enumerate(incidents):
        for resource in set(resources):
            if resource in first_incident_by_resource:
                dsu.union(index, first_incident_by_resource[resource])
            else:
                first_incident_by_resource[resource] = index

    groups = defaultdict(list)
    for index, incident_id in enumerate(incident_ids):
        groups[dsu.find(index)].append(incident_id)

    result = [sorted(group) for group in groups.values()]
    return sorted(result, key=lambda group: group[0])


incidents = [
    ("INC-1", ["node-a", "service-payments"]),
    ("INC-2", ["service-payments"]),
    ("INC-3", ["node-z"]),
    ("INC-4", ["node-a"]),
    ("INC-5", []),
]
assert correlate_incidents(incidents) == [
    ["INC-1", "INC-2", "INC-4"],
    ["INC-3"],
    ["INC-5"],
]
assert correlate_incidents([]) == []
assert correlate_incidents([("INC-1", []), ("INC-2", [])]) == [
    ["INC-1"], ["INC-2"]
]
~~~

### Why it works

Every incident begins in its own component. The first incident associated with a resource is remembered. Each later incident that references that resource is unioned with the first one. Repeated unions connect incidents linked through chains of shared resources. Incidents without resources remain singleton groups.

Let I be the number of incidents and R the total number of resource references:

- **Time:** O((I + R) α(I)) amortized for union/find, plus sorting the resulting IDs and groups.
- **Auxiliary space:** O(I + U), where U is the number of distinct resource identifiers.

**Important limitation:** Shared infrastructure is evidence of a relationship, not proof of a common root cause. Correlation helps organize investigation; it does not establish causality.

---

## Operational concepts and algorithm boundaries

| Platform concept | What the official source establishes | What this repository demonstrates |
|---|---|---|
| Kubernetes Pod lifecycle | Pods have lifecycle phases, conditions, and container health probes. | Simplified event and dependency analysis; not a Pod controller. |
| Kubernetes Deployments | A Deployment manages Pods and ReplicaSets and reconciles declared desired state toward actual state at a controlled rate. | Scheduling and capacity exercises; not the Kubernetes rollout algorithm. |
| Prometheus data model | Time series are identified by metric name and label dimensions; changing a label value creates a different time series. | In-memory grouping of sample event records; not Prometheus storage or PromQL execution. |
| PromQL queries | Prometheus supports instant and range queries over time-series data. | A simple rolling-window calculation with an explicitly defined event boundary. |

## Testing checklist

Test more than the happy path:

- Duplicate and malformed event records.
- Missing or unknown service/resource identifiers.
- Equal timestamps and exact cooldown/window boundaries.
- Empty streams and streams with only successes or only failures.
- Retry limits, retries beyond the horizon, and simultaneous retry times.
- Dependency graphs with disconnected nodes and cycles.
- Intervals that touch but do not overlap.
- Error budgets that are exactly exhausted or exceeded.
- Repeated resources and incidents with no resource references.
- Large inputs where a repeated full scan would be unnecessarily expensive.

For every time-based algorithm, write down the boundary rule before implementing it. Terms such as "last five minutes", "cooldown", and "at the same time" are not precise enough without a convention.

## Suggested practice schedule

| Session | Work |
|---|---|
| 1 | Failure-signature aggregation and rolling failure rates |
| 2 | Token bucket and retry scheduling |
| 3 | Dependency blast-radius analysis and critical path |
| 4 | Peak concurrency and interval boundary tests |
| 5 | Error-budget calculations and alert cooldown |
| 6 | Incident correlation with disjoint sets |
| 7 | Re-implement two problems and explain their production limitations |

Change one assumption and explain what must change in the algorithm: out-of-order events, per-tenant token buckets, retry jitter, dependency cycles, finite worker capacity, or alert suppression shared across processors.

## Completion checklist

- [ ] I can translate an operational story into explicit input and output contracts.
- [ ] I can define timestamp ordering and interval boundary semantics.
- [ ] I can select a counter, deque, heap, graph, or disjoint-set structure for a reason.
- [ ] I can explain the invariant maintained by the selected structure.
- [ ] I can distinguish a simulation from actual Kubernetes or Prometheus behavior.
- [ ] I can explain why a dependency graph does not prove root cause.
- [ ] I can state the assumptions behind an error-budget calculation.
- [ ] I can test boundary conditions and invalid inputs.
- [ ] I can discuss concurrency, persistence, distributed coordination, and failure modes omitted by the exercise.
- [ ] I can explain time and auxiliary-space complexity.

## Navigation

- [DSA Interview Preparation — Main Guide](../README.md)
- [01 — Interview Roadmap](../01%20Interview%20Roadmap/README.md)
- [03 — Pattern Recognition](../03%20Pattern%20Recognition/README.md)
- [04 — Easy Problem Sets](../04%20Easy%20Problem%20Sets/README.md)
- [05 — Medium Problem Sets](../05%20Medium%20Problem%20Sets/README.md)
- [06 — Advanced Problem Sets](../06%20Advanced%20Problem%20Sets/README.md)
- [DSA for DevOps with Python](../../DSA%20for%20DevOps/README.md)
- [Complete DSA — Interview Patterns](../../Complete%20DSA/17%20Interview%20Patterns/README.md)

## Authoritative references

- [Kubernetes — Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Kubernetes — Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Prometheus — Data Model](https://prometheus.io/docs/concepts/data_model/)
- [Prometheus — Querying Basics](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Python — collections](https://docs.python.org/3/library/collections.html)
- [Python — heapq](https://docs.python.org/3/library/heapq.html)
- [Python — unittest](https://docs.python.org/3/library/unittest.html)

The platform references establish the documented concepts summarized above. Problem contracts, example data, retry policy, cooldown policy, error-budget arithmetic, and Python implementations are educational repository design decisions, not official Kubernetes, Prometheus, or SRE product behavior.
