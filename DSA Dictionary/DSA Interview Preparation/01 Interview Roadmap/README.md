# 01 — Interview Roadmap

This guide helps you turn DSA study into a repeatable interview-preparation plan. It explains where to start, what to study, how to practice, how to review mistakes, and how to decide when to move to the next stage.

It is a **study framework**, not a promise that a fixed number of weeks or solved problems will make someone pass every interview. Hiring processes differ by company, role, seniority, and interview format.

---

## 1. What you should be able to do

The goal is not simply to memorize algorithms. By the time you are ready to attempt a coding interview, you should be able to:

1. Understand a problem statement and identify missing assumptions.
2. Convert examples and constraints into a clear input/output contract.
3. Build a correct straightforward solution.
4. Recognize when a data structure or problem-solving pattern can improve that solution.
5. Explain why the approach works.
6. Write readable Python without relying on copied code.
7. Test normal cases, boundary cases, and cases likely to expose mistakes.
8. Analyze time and auxiliary-space complexity.
9. Explain trade-offs and adapt when a requirement changes.
10. Communicate your reasoning while managing limited time.

These are preparation goals defined for this repository. They are not a universal hiring rubric.

## 2. Start with a self-assessment

Before choosing a schedule, try these activities without reading a solution:

| Skill | Self-check |
|---|---|
| Python basics | Can you write a function using conditions, loops, and return values? |
| Lists and strings | Can you traverse, index, slice, and update a list correctly? |
| Dictionaries and sets | Can you count frequencies, detect duplicates, and perform efficient lookups? |
| Core structures | Can you explain when to use a stack, queue, linked list, tree, heap, or graph? |
| Searching and sorting | Can you explain the requirement that makes binary search applicable? |
| Patterns | Can you recognize a likely two-pointer, sliding-window, prefix-sum, or traversal approach? |
| Complexity | Can you explain why a loop is linear or why nested loops may be quadratic? |
| Testing | Can you construct tests beyond the sample input? |
| Communication | Can you explain your approach before and during implementation? |

Mark each item:

- **Not started:** I have not studied this yet.
- **Learning:** I understand it with notes or examples.
- **Practicing:** I can solve familiar exercises independently.
- **Reliable:** I can explain and adapt it without depending on a reference.

Be honest. A realistic starting point is more useful than an optimistic schedule.

## 3. Recommended learning sequence

Follow the stages in order, but spend more time on areas where your self-assessment shows a gap.

### Stage 1 — Python and problem-solving foundations

Study:

- Variables, conditions, loops, functions, and return values.
- Lists, strings, dictionaries, and sets.
- Iteration, indexing, slicing, and basic input validation.
- Tracing code by hand.
- Basic time and space complexity.
- Writing small tests and interpreting failures.

**Practice outcome:** You can implement a small function, trace it with an example, and test ordinary and boundary inputs.

If you need a beginner-friendly start, use [DSA for DevOps with Python](../../DSA%20for%20DevOps/README.md), especially its Python basics, lists, search, counting, sets, and dictionaries lessons.

### Stage 2 — Core data structures

Study:

- Arrays and strings.
- Hash maps and sets.
- Linked lists.
- Stacks and queues.
- Trees and binary search trees.
- Heaps and priority queues.
- Graph representations.

For each structure, understand the operations it supports, the costs relevant to the problem, and the cases where it is a poor fit.

**Practice outcome:** Given a problem's operations and constraints, you can explain which structure you would consider and why.

Use the matching chapters in [Complete DSA](../../Complete%20DSA/README.md) as the concept reference.

### Stage 3 — Essential problem-solving patterns

Practice recognizing and applying:

- Two pointers.
- Fast and slow pointers.
- Sliding window.
- Prefix sums and frequency counting.
- Binary search and boundary search.
- Sorting and interval merging.
- Stack and monotonic-stack patterns.
- Heap-based top-K or priority problems.
- BFS and DFS.
- Topological sorting.
- Recursion and backtracking.
- Greedy reasoning.
- Dynamic programming.

Do not memorize a signal as a rule. For example, a contiguous range may suggest a sliding window, but the constraints and value properties determine whether a particular window strategy is valid.

**Practice outcome:** You can identify a candidate pattern, explain the property that supports it, and reject it when its assumptions do not hold.

Use [Complete DSA — Interview Patterns](../../Complete%20DSA/17%20Interview%20Patterns/README.md) for a pattern recognition reference.

### Stage 4 — Mixed problem practice

Move beyond solving several problems of the same type in a row.

- Mix arrays, strings, maps, stacks, queues, trees, and graphs.
- Read the problem before deciding which topic it belongs to.
- Solve without looking at the chapter title or pattern label.
- Compare a straightforward approach with a more efficient one.
- Revisit previously solved problems after a delay.

**Practice outcome:** You can choose an approach from the problem itself rather than from a topic hint.

Use [Complete DSA — Practice Problems](../../Complete%20DSA/18%20DSA%20Practice%20Problems/README.md) as one source of exercises.

### Stage 5 — DevOps and SRE-oriented coding

Apply the same DSA skills to simplified operational problem statements, for example:

- Count event types in a collection of log records.
- Detect duplicate host or resource identifiers.
- Process jobs in arrival order or by priority.
- Traverse a service-dependency graph.
- Find reachability or a path in a simplified topology.
- Maintain a bounded collection of recent events.

Treat these as coding exercises, not production architecture. A real monitoring, deployment, inventory, or orchestration integration has additional requirements such as data contracts, failure handling, security, concurrency, and observability.

**Practice outcome:** You can map an operational story to inputs, outputs, constraints, a suitable data structure, and test cases.

### Stage 6 — Timed practice and interview simulation

Only add strict time limits after you can solve the relevant problem types accurately without a timer.

Practice the complete interview loop:

1. Read and clarify.
2. State assumptions.
3. Work through an example.
4. Explain a baseline.
5. Choose and justify an approach.
6. Implement.
7. Test.
8. Analyze complexity.
9. Discuss a follow-up.

**Practice outcome:** You can manage the process and communicate clearly even when the problem is unfamiliar.

---

## 4. Choose a study schedule

The following schedules are repository planning options, not externally validated guarantees. Pick the one that fits your available time and current skill level. If a stage is not reliable yet, extend it rather than skipping it to meet a calendar date.

### Option A — Four-week focused review

Best treated as a refresher for someone who already knows core DSA.

| Week | Main focus | Suggested work |
|---|---|---|
| 1 | Python fluency and core structures | Review lists, strings, maps, sets, stacks, queues, linked lists |
| 2 | Patterns | Practice two pointers, sliding window, prefix sums, binary search, intervals, heaps |
| 3 | Trees, graphs, recursion, greedy, DP | Review traversals and solve mixed exercises |
| 4 | Interview execution | Timed practice, debugging, revision, and mock interviews |

This is a compact review plan. It is not a realistic assumption for every beginner.

### Option B — Eight-week structured preparation

A balanced plan for someone with basic Python and some DSA familiarity.

| Week | Main focus | Suggested work |
|---|---|---|
| 1 | Python, complexity, arrays, strings | Rebuild fundamentals and test edge cases |
| 2 | Hash maps, sets, linked lists | Counting, lookup, duplicates, pointer manipulation |
| 3 | Stacks, queues, searching, sorting | Boundary cases, monotonic structures, binary search |
| 4 | Two pointers, sliding window, prefix sums | Practice recognizing contiguous-range and pair problems |
| 5 | Trees and heaps | Traversals, search properties, priority-based problems |
| 6 | Graphs and dependency problems | BFS, DFS, components, cycle reasoning, topological ordering |
| 7 | Greedy, backtracking, dynamic programming | Learn state/choice reasoning and compare alternatives |
| 8 | Mixed practice and interview simulation | Timed sets, review weak areas, and complete mock sessions |

### Option C — Twelve-week gradual preparation

A more flexible option when you are learning DSA while working or when several fundamentals are new.

| Week | Main focus |
|---|---|
| 1 | Python foundations and tracing |
| 2 | Complexity, arrays, and strings |
| 3 | Dictionaries, sets, and frequency counting |
| 4 | Linked lists, stacks, and queues |
| 5 | Sorting, searching, and binary search |
| 6 | Two pointers, sliding window, and prefix sums |
| 7 | Trees and binary search trees |
| 8 | Heaps and priority queues |
| 9 | Graphs, BFS, DFS, and dependencies |
| 10 | Greedy, recursion, and backtracking |
| 11 | Dynamic programming and mixed problem solving |
| 12 | DevOps/SRE scenarios, timed assessments, mocks, and revision |

You can extend this plan. Understanding and independent implementation matter more than finishing a week on a particular date.

## 5. A repeatable weekly routine

Use a routine that includes learning, solving, testing, and review.

| Session type | What to do |
|---|---|
| Concept review | Read one concept and write a short explanation in your own words |
| Guided implementation | Implement a small example and trace it manually |
| Independent problem | Attempt a problem before opening hints or solutions |
| Testing and debugging | Add edge cases, inspect failures, and correct the implementation |
| Mixed revision | Re-solve an older problem without looking at your previous code |
| Communication practice | Explain one solution aloud, including complexity and trade-offs |

A sample weekly rhythm could be:

- **Early sessions:** Learn a concept and implement its basic operations.
- **Middle sessions:** Solve focused exercises and test edge cases.
- **Later sessions:** Solve mixed problems without topic labels.
- **End-of-week review:** Record mistakes and choose the next weak area.

Adjust the number and length of sessions to your actual availability. Do not treat this sample rhythm as a required daily schedule.

## 6. How to practice one problem

Use the same process each time:

### Before coding

- Restate the task.
- Identify the input and expected output.
- Read every constraint.
- Ask about ambiguity rather than silently assuming behavior.
- Work through a normal example and a boundary example.

### While designing

- Describe a straightforward correct approach.
- Estimate its time and auxiliary-space costs.
- Identify the expensive operation or repeated work.
- Consider whether a known data structure or pattern addresses that cost.
- State the invariant or reasoning behind the improved approach.

### While implementing

- Use clear names.
- Keep the implementation focused on the stated contract.
- Handle empty and smallest valid inputs.
- Avoid changing input data unless allowed.
- Do not rely only on the provided example.

### After implementing

- Test normal, boundary, duplicate, and adversarial cases where relevant.
- Explain the result of each important step.
- State time and auxiliary-space complexity.
- Mention trade-offs and possible follow-ups.
- Record what you would do differently next time.

The [Interview Patterns chapter](../../Complete%20DSA/17%20Interview%20Patterns/README.md) provides a more detailed repeatable workflow.

## 7. Keep a problem-solving tracker

Maintain a simple table in your notes or spreadsheet. One row should represent one problem attempt, not just one problem name.

| Date | Problem | Topic/pattern | Independent? | Main mistake | Revisit |
|---|---|---|---|---|---|
| YYYY-MM-DD | Example problem | Hash map | Yes / With hint / No | Missed duplicate case | YYYY-MM-DD |

Use consistent labels:

- **Independent:** Solved without hints or solution references.
- **With hint:** Needed a hint but completed the reasoning and implementation.
- **Reviewed:** Read or studied a solution but did not independently solve it.
- **Re-solved:** Solved again later without relying on the previous implementation.

Do not count a copied or heavily guided solution as an independent solve. The tracker should show what you can actually reproduce.

## 8. How to decide what to study next

At the end of each review cycle, classify your gaps.

| Observation | Next action |
|---|---|
| You cannot explain the data structure | Return to the relevant Complete DSA chapter |
| You understand the concept but cannot implement it | Write small implementations before attempting more problems |
| You can implement but cannot recognize when to use it | Practice mixed problems without topic labels |
| Your approach works but is too slow | Compare the baseline with a suitable optimized approach |
| You miss boundary cases | Build a deliberate edge-case checklist for that problem type |
| You cannot explain complexity | Count the operations and identify the input-size variables |
| You solve correctly but struggle with time | Practice the interview workflow under a moderate time limit |
| You stop communicating while coding | Practice explaining decisions aloud while solving |

Do not respond to every failure by learning a new algorithm. Sometimes the missing skill is careful reading, tracing, testing, or explanation.

## 9. Readiness review

You are making useful progress when you can consistently:

- Solve foundational problems without copying code.
- Select data structures based on required operations.
- Recognize common patterns in mixed problem sets.
- Explain why the chosen approach is correct.
- Test edge cases without being prompted.
- Analyze time and auxiliary space.
- Debug your own implementation.
- Explain trade-offs and respond to changed constraints.
- Communicate clearly while working.

A checklist is more useful than a single problem-count target. If several items remain weak, continue practicing those areas before relying heavily on timed assessments.

For a final review, use the **12 Interview Readiness Checklist** section when it is available in this track.

## 10. Official Python references

These official references support the Python prerequisites and testing practices used in this roadmap:

- [The Python Tutorial](https://docs.python.org/3/tutorial/index.html) — introduces Python's language concepts and expects readers to have basic programming understanding.
- [Python Tutorial — Data Structures](https://docs.python.org/3/tutorial/datastructures.html) — lists, list methods, stacks, and queues.
- [unittest — Unit testing framework](https://docs.python.org/3/library/unittest.html) — test cases, fixtures, suites, runners, and assertions.

The schedules, stages, tracker format, and readiness signals in this document are **repository study-design decisions**, not official Python guidance or a hiring standard.

---

## Navigation

- [Back to DSA Interview Preparation](../README.md)
- [DSA for DevOps with Python](../../DSA%20for%20DevOps/README.md)
- [Complete DSA](../../Complete%20DSA/README.md)
- [Complete DSA — Interview Patterns](../../Complete%20DSA/17%20Interview%20Patterns/README.md)
- [Complete DSA — Practice Problems](../../Complete%20DSA/18%20DSA%20Practice%20Problems/README.md)

**Next:** 02 Python Coding Interview.
