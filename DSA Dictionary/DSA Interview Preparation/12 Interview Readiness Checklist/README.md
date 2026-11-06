# 12 — Interview Readiness Checklist

Use this checklist to decide what to practise before, during, and after a coding interview. It brings together the skills developed across the DSA Interview Preparation track: Python fluency, data structures, pattern recognition, correctness, testing, complexity analysis, debugging, communication, and timed practice.

This guide is intended for DevOps, SRE, platform, cloud, and general software engineering candidates who may encounter Python coding interviews.

> **Important:** This is a self-assessment and preparation aid, not an official hiring rubric or a guarantee of interview success. Interview formats and expectations vary by employer, role, seniority, and interviewer. Prioritize the skills that are relevant to the actual job description and interview process.

## Table of contents

1. [How to use this checklist](#1-how-to-use-this-checklist)
2. [Understand the role and interview process](#2-understand-the-role-and-interview-process)
3. [Python coding readiness](#3-python-coding-readiness)
4. [Data structures and algorithms readiness](#4-data-structures-and-algorithms-readiness)
5. [Problem-solving and communication readiness](#5-problem-solving-and-communication-readiness)
6. [Correctness, testing, and debugging readiness](#6-correctness-testing-and-debugging-readiness)
7. [Complexity and optimization readiness](#7-complexity-and-optimization-readiness)
8. [DevOps and SRE coding readiness](#8-devops-and-sre-coding-readiness)
9. [Timed assessment and mock interview readiness](#9-timed-assessment-and-mock-interview-readiness)
10. [Before, during, and after the interview](#10-before-during-and-after-the-interview)
11. [Final self-assessment](#11-final-self-assessment)
12. [A focused final-week plan](#12-a-focused-final-week-plan)
13. [Common readiness traps](#13-common-readiness-traps)
14. [References and navigation](#14-references-and-navigation)

---

## 1. How to use this checklist

Do not try to tick every box in one sitting. Use the checklist at three points:

- **Preparation review:** Identify gaps before scheduling intensive practice.
- **Final review:** Check the skills you can demonstrate without notes.
- **Post-interview review:** Record what happened and update your revision plan.

### Four evidence labels

For each skill, use one of these labels:

| Status | Meaning |
|---|---|
| Not assessed | You have not tried this skill yet |
| Needs work | You cannot yet demonstrate it reliably |
| Developing | You can do it with notes, hints, or extra time |
| Consistent in practice | You have demonstrated it independently across multiple exercises |

“Consistent in practice” is a self-review label. It does not predict an employer's decision.

### Use evidence, not confidence alone

A useful readiness claim should have evidence behind it:

- “I understand sliding window” → I can state the invariant and solve a new sliding-window problem.
- “I know graph algorithms” → I can model dependencies, choose traversal or topological sorting, detect cycles when required, and test disconnected components.
- “I can code in Python” → I can implement a solution without relying on copied snippets and explain the language features I use.
- “I am interview-ready” → I have completed timed practice and mock interviews, reviewed mistakes, and tested myself on unfamiliar problems.

### Do not confuse DSA readiness with total role readiness

This checklist focuses on coding and algorithmic interview preparation. A DevOps, SRE, platform, or cloud role may also assess Linux, networking, cloud services, containers, Kubernetes, infrastructure as code, CI/CD, observability, incident response, security, and system design. Check the actual role description and prepare those areas separately.

## 2. Understand the role and interview process

Before spending all your preparation time on algorithms, establish what the target role actually requires.

### Role research

- [ ] I have read the job description carefully.
- [ ] I can explain the role's main responsibilities in my own words.
- [ ] I have identified the programming language(s) expected.
- [ ] I have identified whether the role is primarily software engineering, DevOps, SRE, platform, cloud, or a combination.
- [ ] I have listed the technical areas explicitly mentioned in the job description.
- [ ] I have separated required skills from preferred skills.
- [ ] I have identified the seniority level and expected scope of ownership.
- [ ] I have prepared examples from my own work that demonstrate relevant engineering skills.

### Interview-process clarification

Where appropriate, ask the recruiter or hiring contact about:

- [ ] Number and type of interview stages.
- [ ] Whether a coding exercise is included.
- [ ] Whether the coding language is flexible.
- [ ] Whether the environment is a shared editor, online assessment, take-home task, or another format.
- [ ] Whether execution, tests, and standard-library documentation are available.
- [ ] Whether the role includes system design, troubleshooting, or operational scenarios.
- [ ] Whether there are role-specific topics to prepare.

Do not assume that every employer uses the same interview format. If a detail is not provided, record it as unknown rather than treating an assumption as fact.

### Role-specific preparation map

| Role focus | DSA preparation | Additional areas to review |
|---|---|---|
| General software engineering | Core data structures, algorithms, problem patterns, coding quality | Design, APIs, testing, language fundamentals |
| DevOps engineering | Strings, arrays, maps, heaps, graphs, intervals, parsing | Linux, networking, Git, CI/CD, containers, cloud, automation |
| SRE | Queues, windows, graphs, scheduling, rate and error calculations | Reliability, observability, incident response, capacity, distributed systems |
| Platform engineering | Graphs, dependency modeling, scheduling, data transformation | Kubernetes, APIs, automation, infrastructure as code, platform design |
| Cloud engineering | Data handling, graph traversal, scheduling, resource calculations | Cloud services, IAM, networking, resiliency, cost, security |

This table is a repository planning aid, not a claim that every employer tests every listed topic.

## 3. Python coding readiness

You should be able to write and explain ordinary Python without spending most of the interview recalling syntax.

### Language fundamentals

- [ ] I can write functions with clear inputs and return values.
- [ ] I can use conditions, loops, and comprehensions appropriately.
- [ ] I understand local variables, scope, and return behavior.
- [ ] I can work with strings, lists, tuples, dictionaries, and sets.
- [ ] I understand mutability and the difference between modifying an object and creating a new one.
- [ ] I avoid mutable default arguments when a fresh object is required for each call.
- [ ] I can use exceptions for invalid input or exceptional conditions when the contract requires it.
- [ ] I can read a traceback and locate the relevant source line.
- [ ] I can write readable code with meaningful names.
- [ ] I can explain any type annotations I choose to use.

### Built-in and standard-library fluency

- [ ] I know when a list, set, tuple, or dictionary is appropriate.
- [ ] I can use enumerate and zip where they make iteration clearer.
- [ ] I can use sorted and list.sort while understanding whether a new list is produced or the existing list is modified.
- [ ] I can use Counter for frequency counting.
- [ ] I can use defaultdict when a default value per missing key is useful.
- [ ] I can use deque for efficient operations at both ends.
- [ ] I can use heapq for priority-queue operations.
- [ ] I can use set operations for membership and uniqueness tasks.
- [ ] I can use unittest or another agreed test framework to verify behavior.
- [ ] I know that type annotations are not automatically enforced by the Python runtime.

### Python interview pitfalls

- [ ] I do not mutate a collection while iterating over it unless the behavior is deliberate and safe.
- [ ] I do not assume dictionary or set operations have a strict worst-case constant-time guarantee.
- [ ] I handle empty collections and missing keys explicitly.
- [ ] I understand shallow copying versus shared nested objects when it matters.
- [ ] I do not use broad exception handling to hide programming errors.
- [ ] I avoid clever one-liners that make correctness harder to inspect.
- [ ] I do not claim a function is type-safe merely because it has annotations.
- [ ] I can explain whether a function mutates its inputs.

### Python practical check

Without looking at notes, write small functions that:

1. Count the frequency of values in a list.
2. Remove duplicates while preserving first-occurrence order.
3. Group records by a dictionary key.
4. Return the k largest values using an appropriate approach.
5. Read a CSV file using the standard library.
6. Write unit tests for empty, ordinary, and invalid inputs.

If you cannot complete these comfortably, revisit [02 — Python Coding Interview](../02%20Python%20Coding%20Interview/README.md).

## 4. Data structures and algorithms readiness

The goal is not to memorize every algorithm. You should understand the common structures, recognize their operations, and select a suitable approach for the problem constraints.

### Core data structures

- [ ] Arrays and strings: indexing, traversal, slicing trade-offs, and in-place updates.
- [ ] Hash maps and sets: counting, lookup, uniqueness, and expected-time assumptions.
- [ ] Linked lists: pointer movement, reversal, cycle detection, and dummy nodes.
- [ ] Stacks: last-in-first-out behavior and monotonic-stack applications.
- [ ] Queues and deques: first-in-first-out behavior and sliding-window use.
- [ ] Heaps: priority queues, top-K, and repeated minimum/maximum extraction.
- [ ] Trees: traversal, recursion, depth, and binary search tree properties.
- [ ] Graphs: adjacency lists, traversal, visited state, and disconnected components.
- [ ] Disjoint Set Union: connectivity and component merging.
- [ ] Tries: prefix lookup and string-path representation.

### Core algorithms and patterns

- [ ] Sorting and comparator/key-based ordering.
- [ ] Linear search and binary search.
- [ ] Two pointers.
- [ ] Sliding window.
- [ ] Prefix sums.
- [ ] Fast and slow pointers.
- [ ] Merge intervals and sweep-line reasoning.
- [ ] Monotonic stack and deque.
- [ ] Breadth-first search.
- [ ] Depth-first search.
- [ ] Topological sorting and cycle detection.
- [ ] Shortest paths at the level relevant to the target role.
- [ ] Backtracking.
- [ ] Greedy algorithms.
- [ ] Dynamic programming fundamentals.

### Pattern selection

Before coding, ask:

- [ ] Does the task require fast membership or frequency lookup?
- [ ] Is the input sorted, or can sorting help?
- [ ] Is there a contiguous range or window?
- [ ] Does the problem involve a pair, range, or moving boundary?
- [ ] Is there a dependency relationship or graph?
- [ ] Do I repeatedly need the smallest or largest item?
- [ ] Is the task asking for all combinations or a constrained search?
- [ ] Does the problem have overlapping subproblems and reusable state?
- [ ] What input constraints make a straightforward approach too expensive?

### Minimum practice evidence

For each major pattern you expect to encounter:

- [ ] I can explain the pattern in plain language.
- [ ] I can identify the state the algorithm maintains.
- [ ] I can state the key invariant or correctness argument.
- [ ] I can implement a familiar example without notes.
- [ ] I can solve at least one unfamiliar variation.
- [ ] I can identify common boundary cases.
- [ ] I can explain time and auxiliary-space complexity.

Use [03 — Pattern Recognition](../03%20Pattern%20Recognition/README.md) to practise choosing patterns rather than memorizing problem names.

## 5. Problem-solving and communication readiness

A correct solution is important, but the interviewer also needs to understand how you reached it.

### Understanding the prompt

- [ ] I restate the problem before coding.
- [ ] I identify the expected input and output.
- [ ] I clarify whether the input may be empty.
- [ ] I ask about duplicates, ordering, and negative or boundary values when relevant.
- [ ] I clarify whether mutation is allowed.
- [ ] I identify invalid-input behavior if it affects the solution.
- [ ] I use the constraints to guide the algorithm choice.

### Explaining the approach

- [ ] I can describe a simple correct approach first.
- [ ] I can explain why I chose a data structure.
- [ ] I can compare the straightforward and optimized approaches.
- [ ] I can trace a small example by hand.
- [ ] I can state the invariant or correctness reasoning.
- [ ] I can identify what could make my approach fail.
- [ ] I can explain trade-offs without claiming an optimization is always better.

### Communicating while coding

- [ ] I explain important decisions without narrating every keystroke.
- [ ] I say when I am making an assumption.
- [ ] I ask a focused question when the prompt is ambiguous.
- [ ] I tell the interviewer when I have found a bug and how I will investigate it.
- [ ] I do not pretend that unexecuted code has passed tests.
- [ ] I respond to hints constructively and adapt the approach when needed.

### Explaining the result

- [ ] I summarize what the implementation does.
- [ ] I walk through at least one representative input.
- [ ] I state time complexity and define the variables.
- [ ] I state auxiliary-space complexity separately from output space where useful.
- [ ] I identify important limitations.
- [ ] I answer follow-up questions by revisiting the contract and invariants.

## 6. Correctness, testing, and debugging readiness

Do not stop when the supplied example works. Test the contract.

### Test design

- [ ] I test a normal input.
- [ ] I test empty input when it is permitted.
- [ ] I test the smallest meaningful input.
- [ ] I test duplicates where relevant.
- [ ] I test already sorted and reverse-sorted input where relevant.
- [ ] I test negative, zero, or boundary values where relevant.
- [ ] I test disconnected graphs and cycles where relevant.
- [ ] I test equality at interval or time-window boundaries.
- [ ] I test invalid input if the contract defines its behavior.
- [ ] I can explain why each test is useful.

### Debugging workflow

- [ ] I reproduce the failure with the smallest useful input.
- [ ] I identify the expected and actual results.
- [ ] I trace the state that should change.
- [ ] I check loop boundaries and update order.
- [ ] I inspect assumptions about ordering, uniqueness, and mutation.
- [ ] I make one targeted change at a time.
- [ ] I rerun the failing test and relevant regression tests.
- [ ] I can use a debugger or print/logging statements deliberately.
- [ ] I remove temporary debugging output before presenting the final solution.

### Test automation

- [ ] I can write a small unit test.
- [ ] I can group related test cases.
- [ ] I know how to run the tests in the available environment.
- [ ] I can distinguish a test assertion from a manual trace.
- [ ] I can state whether code was actually executed.

The official [unittest documentation](https://docs.python.org/3/library/unittest.html) describes Python's test cases, fixtures, suites, and runners. The [pdb documentation](https://docs.python.org/3/library/pdb.html) describes the interactive debugger, including breakpoints and stepping.

## 7. Complexity and optimization readiness

You should be able to justify the cost of your solution rather than simply naming a Big-O expression.

### Complexity analysis

- [ ] I can define n, V, E, or other variables used in the analysis.
- [ ] I can count the dominant loops and operations.
- [ ] I can recognize nested loops and explain when they do or do not imply quadratic work.
- [ ] I can analyze sorting costs at the appropriate level.
- [ ] I can explain expected-time hash-table assumptions.
- [ ] I can analyze heap insertion and removal costs.
- [ ] I can analyze graph traversal in terms of vertices and edges.
- [ ] I can distinguish auxiliary space from output space.
- [ ] I can explain the cost of recursion or an explicit stack when relevant.
- [ ] I can state whether an analysis is worst-case, average-case, or expected.

### Optimization judgment

- [ ] I establish correctness before optimizing.
- [ ] I identify the actual bottleneck.
- [ ] I consider input constraints before selecting a more complex algorithm.
- [ ] I can explain the memory cost of an optimization.
- [ ] I do not claim a performance improvement without evidence.
- [ ] I know that microbenchmarks can be affected by environment and measurement choices.
- [ ] I can explain why a theoretically better algorithm may not be the best engineering choice for a small input.

Python's [timeit documentation](https://docs.python.org/3/library/timeit.html) describes a tool for timing small snippets while avoiding several common measurement pitfalls. A timing result from one machine is not a universal performance guarantee.

## 8. DevOps and SRE coding readiness

For operationally focused roles, practise translating a real-world description into a precise algorithmic contract. These exercises are simplified models; passing them does not establish production-system expertise by itself.

### Operational problem patterns

- [ ] Parse and aggregate structured log records.
- [ ] Count frequent error signatures.
- [ ] Calculate a rolling rate over a defined time interval.
- [ ] Handle interval boundaries precisely.
- [ ] Schedule work using a priority queue.
- [ ] Model service dependencies as a directed graph.
- [ ] Detect dependency cycles.
- [ ] Calculate connected components or affected services.
- [ ] Simulate a token bucket or cooldown policy.
- [ ] Calculate a simple error budget from explicitly defined inputs.
- [ ] Group incidents by shared resources.
- [ ] Explain the limits of an in-memory simulation.

### Production-awareness questions

For an operational coding problem, ask:

- [ ] What happens if events arrive late or out of order?
- [ ] Can messages be duplicated?
- [ ] What happens when a process restarts?
- [ ] Is state local to one worker or shared?
- [ ] Can multiple workers process the same item?
- [ ] What happens if a dependency is unavailable?
- [ ] Is retry behavior bounded?
- [ ] How are timeouts and cancellation handled?
- [ ] What telemetry would help diagnose failures?
- [ ] What security or data-retention constraints apply?

You do not need to implement a distributed production platform in a short algorithm interview. You should, however, be able to state which production concerns the exercise intentionally leaves out.

Review [07 — DevOps and SRE Problems](../07%20DevOps%20and%20SRE%20Problems/README.md) for operationally themed exercises.

## 9. Timed assessment and mock interview readiness

### Timed practice

- [ ] I have completed at least one timed set without reading solutions.
- [ ] I can allocate time between understanding, design, coding, and testing.
- [ ] I know when to simplify an approach to secure a correct baseline.
- [ ] I can decide when to move on from a blocked approach.
- [ ] I reserve time to test and explain complexity.
- [ ] I record incomplete work honestly.
- [ ] I review mistakes after the timed session rather than only checking the score.

### Mock interview

- [ ] I have completed a mock with another person or in solo simulation.
- [ ] I have practised speaking through the approach.
- [ ] I have answered clarifying and follow-up questions.
- [ ] I have received or recorded feedback on communication.
- [ ] I have reviewed at least one implementation under time pressure.
- [ ] I can identify two strengths and two improvement areas from a mock.
- [ ] I have converted feedback into a concrete next action.

Suggested practice sessions and scoring rubrics in this repository are educational design choices, not employer standards.

Continue with [09 — Timed Coding Assessments](../09%20Timed%20Coding%20Assessments/README.md) and [10 — Mock Interviews](../10%20Mock%20Interviews/README.md).

## 10. Before, during, and after the interview

### Before the interview

**Preparation**
- [ ] Re-read the job description and your notes about the interview stages.
- [ ] Review the language and tools you intend to use.
- [ ] Revise a small number of weak patterns; do not attempt to learn an entire advanced topic at the last minute.
- [ ] Complete a short warm-up problem.
- [ ] Review your own previous mistakes and the fixes you validated.
- [ ] Prepare concise examples of relevant engineering work.
- [ ] Prepare questions about the team, role, and engineering practices.

**Environment**
- [ ] Confirm the interview time and time zone.
- [ ] Confirm the meeting link or assessment access.
- [ ] Check your internet connection, microphone, and camera if needed.
- [ ] Check power, battery, and a quiet working location.
- [ ] Confirm the coding environment and allowed resources if this information is available.
- [ ] Have a permitted way to take notes.
- [ ] Close unrelated applications and notifications where practical.

**Professional preparation**
- [ ] Be ready to introduce your background briefly.
- [ ] Be prepared to explain your own projects and decisions accurately.
- [ ] Do not claim ownership of work you did not do.
- [ ] Do not disclose confidential employer or customer information.

### During the interview

- [ ] Listen to the complete prompt.
- [ ] Restate the task and clarify assumptions.
- [ ] Explain a straightforward approach.
- [ ] Discuss complexity before or after implementation as appropriate.
- [ ] Write readable code.
- [ ] Test meaningful cases.
- [ ] Communicate blockers honestly.
- [ ] Respond constructively to feedback and hints.
- [ ] Keep the interviewer informed when changing direction.
- [ ] Summarize the final solution and limitations.

If you do not know something, say what you do know, explain how you would investigate it, and avoid inventing behavior or APIs.

### After the interview

- [ ] Record the topics and problem patterns while they are still fresh.
- [ ] Note which assumptions or edge cases were difficult.
- [ ] Record whether the solution was completed and tested.
- [ ] Identify one technical improvement and one communication improvement.
- [ ] Add relevant topics to the revision queue.
- [ ] Send any appropriate follow-up through the expected hiring channel.
- [ ] Respect confidentiality and do not publish private interview material.

Do not treat one interview outcome as a complete measure of your engineering ability. Use the experience to improve the next preparation cycle.

## 11. Final self-assessment

Complete this review honestly. Use evidence from recent, independent practice rather than relying on memory of old exercises.

### Readiness worksheet

~~~text
Target role:
Target programming language:
Interview date or preparation window:

PYTHON
Can I implement common operations without notes?
Can I explain the Python features I use?
Can I write and run basic tests?

DSA
Which patterns can I recognize independently?
Which patterns still require hints?
Can I solve unfamiliar variations?

PROBLEM SOLVING
Do I clarify input and output contracts?
Can I explain an invariant or correctness argument?
Can I compare a simple and optimized approach?

QUALITY
Do I test boundary and edge cases?
Can I debug a failing example systematically?
Can I explain time and auxiliary-space complexity?

INTERVIEW PRACTICE
Have I completed a timed assessment?
Have I completed a mock interview?
What feedback did I receive?

ROLE-SPECIFIC
Which non-DSA skills does the job require?
Which of those skills have I demonstrated recently?

TOP THREE GAPS
1.
2.
3.

NEXT ACTIONS
1.
2.
3.
~~~

### Personal go / continue / revise decision

Use these questions to choose your next action:

- **Continue to interviews:** You can solve a range of relevant problems independently, explain your reasoning, test your code, and have practised the interview format you expect.
- **Continue applying while revising:** You have useful foundations but still have gaps. Real interviews can provide additional experience; keep improving in parallel.
- **Focus on targeted revision:** You repeatedly struggle with a prerequisite such as Python fluency, core data structures, or explaining correctness. Return to the relevant lesson and practise a smaller problem before attempting another timed session.

These are personal planning suggestions, not pass/fail thresholds.

## 12. A focused final-week plan

This is an example schedule for someone who already has the foundations. It is not a crash course and should be adjusted to the available time and target role.

| Day | Focus | Suggested activity |
|---|---|---|
| 7 days before | Diagnose gaps | Review attempt log and choose three weak areas |
| 6 days before | Core patterns | Solve one familiar and one unfamiliar problem |
| 5 days before | Python and testing | Implement a small solution and write edge-case tests |
| 4 days before | Role-specific problems | Practise one DevOps/SRE or target-domain exercise |
| 3 days before | Timed practice | Complete a timed assessment and review it |
| 2 days before | Mock interview | Complete one mock and record feedback |
| 1 day before | Light revision | Review notes, environment, schedule, and logistics |
| Interview day | Be prepared and rested | Brief warm-up; avoid last-minute cramming |

If you have less than a week, prioritize the role requirements, a few representative problems, and interview logistics. Do not try to cover every advanced algorithm at the last minute.

## 13. Common readiness traps

### Memorizing solutions

A memorized implementation may fail when the input contract changes. Practise explaining the invariant and solving a different problem with the same pattern.

### Solving only easy problems

Easy exercises build fluency, but they do not demonstrate that you can handle more complex constraints. Include medium-level and role-relevant problems in your practice.

### Solving only advanced problems

Advanced topics do not compensate for weak fundamentals. Make sure you can write clean Python, use common data structures, and test ordinary cases.

### Ignoring communication

A silent solution makes it difficult for an interviewer to understand your decisions. Practise explaining the approach and important state changes clearly.

### Optimizing before understanding

A complex algorithm built on a wrong interpretation is still wrong. Clarify the contract and establish a correct baseline first.

### Testing only the sample

Examples illustrate behavior; they rarely cover every boundary. Design tests from the contract.

### Treating Big-O as a slogan

Explain what n, V, or E represent and which operations dominate. State assumptions such as expected hash-table lookup behavior.

### Confusing a coding exercise with production readiness

An in-memory graph or rate-window simulation does not handle distributed coordination, retries, persistence, observability, security, or partial failures unless those requirements are explicitly included.

### Waiting for perfect readiness

Preparation is ongoing. Apply when your experience and skills are relevant, continue learning, and use feedback responsibly. No checklist can remove uncertainty from an interview process.

## 14. References and navigation

### Official Python documentation

- [The Python Tutorial](https://docs.python.org/3/tutorial/) — language fundamentals, control flow, data structures, and modules. The tutorial is intended for programmers who are new to Python, not people who are new to programming.
- [collections — Container datatypes](https://docs.python.org/3/library/collections.html) — specialized containers such as Counter, deque, and defaultdict.
- [typing — Support for type hints](https://docs.python.org/3/library/typing.html) — type annotations and their intended use. Python does not enforce function and variable annotations at runtime.
- [unittest — Unit testing framework](https://docs.python.org/3/library/unittest.html) — test cases, fixtures, suites, and runners.
- [pdb — The Python Debugger](https://docs.python.org/3/library/pdb.html) — breakpoints, stepping, and stack-frame inspection.
- [timeit — Measure execution time of small code snippets](https://docs.python.org/3/library/timeit.html) — timing small snippets while avoiding common measurement traps.

The linked documentation supports claims about Python language and standard-library behavior. The checklists, suggested preparation activities, and readiness labels are repository design decisions.

### Complete DSA Interview Preparation path

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
- [11 — Revision and Progress Tracking](../11%20Revision%20and%20Progress%20Tracking/README.md)
