# 06 — Recursion and Backtracking

Recursion is a way to solve a problem by defining a solution in terms of smaller instances of the same problem. Backtracking builds on this idea by exploring choices, undoing a choice when it cannot lead to a valid solution, and continuing with another choice.

> **Learning goal:** Learn to design recursive solutions with clear base cases and invariants, then extend that reasoning to systematic search and backtracking.

---

## 1. What Is Recursion?

A recursive function calls itself on a smaller or otherwise simpler version of the problem.

A correct recursive design needs a way to stop.

The two essential parts are:

```text
Base case
    ↓
Stop when the problem is small enough.

Recursive case
    ↓
Reduce the problem and solve the smaller case.
```

Without a valid base case or a path toward it, recursion may not terminate.

---

## 2. A Simple Example

Consider computing a factorial:

```text
n! = n × (n - 1)!
```

with a base case such as:

```text
0! = 1
```

A teaching implementation is:

```python
def factorial(n):
    if n == 0:
        return 1
    return n * factorial(n - 1)
```

The important lesson is the structure:

```text
factorial(4)
→ 4 × factorial(3)
→ 4 × 3 × factorial(2)
→ 4 × 3 × 2 × factorial(1)
→ 4 × 3 × 2 × 1 × factorial(0)
```

The calls then return in reverse order.

---

## 3. Base Cases

The base case defines when recursion stops.

For every recursive algorithm, ask:

1. What is the smallest valid input?
2. What should the answer be for that input?
3. Does every recursive call move toward that condition?

For example, a recursive traversal of a linked structure might stop when the current node is `None`.

> **Common mistake:** Writing the recursive call first and adding a stopping condition later. Design the termination condition explicitly before trusting the recursion.

---

## 4. Recursive Case

The recursive case should make measurable progress toward a base case.

For example:

```python
def countdown(n):
    if n == 0:
        return
    print(n)
    countdown(n - 1)
```

Each call decreases `n` by one.

The reasoning is:

```text
n
↓
n - 1
↓
n - 2
↓
...
↓
0
```

A recursive algorithm should not repeatedly recreate the same state without progress unless the algorithm deliberately handles that state through another mechanism.

---

## 5. The Call Stack

Recursive calls create nested execution states.

Conceptually:

```text
factorial(4)
    factorial(3)
        factorial(2)
            factorial(1)
                factorial(0)
```

The innermost call completes first, then the waiting calls resume.

This is why recursion naturally has a stack-like execution model.

Python also places practical limits on recursive depth. The exact behavior and configured limit should be checked against the Python version and runtime environment being used rather than assumed.

Source: Python's `sys` documentation:
https://docs.python.org/3/library/sys.html#sys.getrecursionlimit

---

## 6. Recursion vs Iteration

Many recursive algorithms can also be expressed iteratively.

| Question | Recursion | Iteration |
|---|---|---|
| Control mechanism | Function calls | Loop/state variables |
| Nested state | Call stack | Explicit variables or data structures |
| Readability | Can closely match recursive definitions | Often straightforward for repeated steps |
| Stack-depth concern | Relevant | Usually avoids recursive call depth |
| Best choice | Depends on the problem | Depends on the problem |

Do not assume recursion is automatically better or worse.

Choose the representation that makes correctness, complexity, and operational constraints clear.

---

## 7. Tracing Recursion

A reliable way to understand recursion is to trace calls and returns separately.

For:

```python
def sum_to_n(n):
    if n == 0:
        return 0
    return n + sum_to_n(n - 1)
```

Trace:

```text
Call phase:

sum_to_n(3)
→ 3 + sum_to_n(2)
→ 3 + 2 + sum_to_n(1)
→ 3 + 2 + 1 + sum_to_n(0)

Return phase:

sum_to_n(0) → 0
sum_to_n(1) → 1
sum_to_n(2) → 3
sum_to_n(3) → 6
```

This two-phase view is especially useful for recursive tree traversal and backtracking.

---

## 8. Recursion on Linked Structures

Recursion can naturally process structures whose elements reference smaller substructures.

For a singly linked list:

```text
A → B → C → None
```

A recursive traversal can reason as:

```text
process A
then recursively process B
```

The base case is reached when there is no current node.

For a linked list with `n` nodes, a traversal that visits each node once has **O(n)** time. A recursive implementation also uses call-stack space proportional to the recursion depth.

The exact depth depends on the algorithm and structure.

---

## 9. Recursion on Trees

Trees are especially natural for recursive reasoning because a node can have smaller child subtrees.

Conceptually:

```text
        A
       / \
      B   C
     / \
    D   E
```

A recursive tree algorithm often follows this structure:

```text
solve(current node)
    solve(left subtree)
    solve(right subtree)
```

The order in which the current node is processed determines the traversal pattern.

Common depth-first traversal orders are:

- preorder
- inorder
- postorder

The next tree chapter will cover these structures in detail.

---

## 10. Divide the Problem Into Subproblems

A useful recursive question is:

> **If I already knew how to solve the smaller problem, what would remain for the current problem?**

For example, suppose a function needs to process a sequence recursively.

You might define:

```text
solve(items, position)
```

and make the recursive case responsible for the remaining positions.

The key is to define exactly what the function means.

For example:

> `solve(items, i)` processes every element from index `i` through the end.

Once this contract is clear, the recursive case becomes much easier to reason about.

---

## 11. Recursive Function Contracts

Before writing a recursive function, write one sentence describing its meaning.

Examples:

```text
factorial(n) returns n!.

depth(node) returns the depth of the subtree rooted at node.

search(node, target) returns whether target exists in the subtree rooted at node.
```

Then define:

```text
Base case:
What is the answer for the smallest state?

Recursive case:
How is the current answer derived from smaller states?
```

This prevents a common mistake: writing code that looks recursive without knowing what each call is supposed to represent.

---

## 12. Backtracking

Backtracking is a systematic search strategy.

A typical pattern is:

```text
Choose
  ↓
Explore
  ↓
Undo choice
  ↓
Try another choice
```

The algorithm explores a decision space while abandoning a partial solution when it cannot produce a valid complete solution.

A generic structure is:

```text
backtrack(state):
    if state is a complete solution:
        record solution
        return

    for each valid choice:
        make choice
        backtrack(new state)
        undo choice
```

The `undo` step is what makes the state reusable for the next branch.

---

## 13. Why Backtracking Works

Imagine choosing values for three positions.

```text
            start
          /       \
       choose A  choose B
        /   \       /   \
       ...  ...   ...  ...
```

Each branch represents a different decision sequence.

If a branch violates a constraint, there is no need to continue exploring that branch.

This is called **pruning**.

The algorithm therefore searches a decision tree while avoiding branches that cannot produce useful results according to the problem's rules.

---

## 14. State and Invariants

Backtracking is much easier when the state has a precise meaning.

For example:

```text
path = the choices selected so far
```

Then the invariant might be:

> Every element currently in `path` is a valid prefix of a candidate solution.

At each recursive call:

1. the state satisfies the invariant
2. one valid choice is added
3. recursion explores the resulting state
4. the choice is removed before trying the next branch

This makes the algorithm easier to test and debug.

---

## 15. Subsets

Generating all subsets is a classic backtracking exercise.

For:

```text
[A, B]
```

the subsets are:

```text
[]
[A]
[B]
[A, B]
```

At each position, a common decision is:

```text
exclude current item
or
include current item
```

That creates a binary decision tree.

With `n` independent include/exclude decisions, there are `2^n` possible subsets.

Therefore, any algorithm that explicitly outputs every subset must account for output size that is already exponential in `n`.

---

## 16. Permutations

A permutation problem asks for different orderings of elements.

For three distinct elements:

```text
[A, B, C]
```

there are:

```text
3! = 6
```

permutations.

A backtracking algorithm can:

1. choose an unused element
2. add it to the current path
3. recurse
4. remove it from the path
5. choose another unused element

The number of permutations grows factorially, so explicit generation becomes expensive quickly.

> **Complexity lesson:** Output size can dominate runtime. Do not describe an algorithm generating all permutations as polynomial-time merely because each recursive step is small.

---

## 17. Combination Problems

Combinations select elements without treating different orders as different solutions.

A common recursive state includes:

```text
start position
current path
```

The `start` position prevents the algorithm from repeatedly choosing earlier elements when order is irrelevant.

The exact state representation depends on whether repetition is allowed and what the problem considers a distinct solution.

Always state those rules before designing the recursion.

---

## 18. Constraint Satisfaction

Backtracking is useful when choices must satisfy constraints.

Examples include:

- placing items without conflicts
- assigning values under restrictions
- constructing valid sequences
- finding paths subject to rules
- solving puzzle-like search spaces

A general approach is:

```text
make a valid partial assignment
↓
check constraints
↓
continue if valid
↓
otherwise backtrack
```

The earlier an invalid state can be detected, the more search can potentially be pruned.

---

## 19. Backtracking vs Brute Force

Backtracking is still a form of exhaustive search in many problems, but it can avoid exploring branches that are already known to be invalid.

Think of the difference as:

```text
Naive brute force:
Generate everything → check afterward

Backtracking:
Build incrementally → reject invalid prefixes early
```

This does not mean backtracking is automatically efficient.

Worst-case search spaces can remain exponential or worse depending on the problem and representation.

Never claim that pruning changes an exponential problem into a polynomial one without a proof or a problem-specific bound.

---

## 20. Memoization Preview

Some recursive problems repeatedly solve the same subproblem.

For example:

```text
solve(A)
├── solve(B)
│   └── solve(D)
└── solve(C)
    └── solve(D)
```

If `solve(D)` produces the same result each time, recomputing it is unnecessary.

**Memoization** stores previously computed results so repeated states can reuse them.

This is an important bridge from recursion to **dynamic programming**.

The dynamic-programming chapter will treat memoization and related techniques in detail.

---

## 21. Recursion and Complexity

Do not determine complexity only by counting the number of lines in a recursive function.

Ask:

1. How many recursive calls does each state create?
2. How does the input size change?
3. How many distinct states are reached?
4. Are states repeated?
5. How much work happens outside recursive calls?
6. How deep can recursion become?

For example, a recurrence may be written conceptually as:

```text
T(n) = T(n - 1) + O(1)
```

which leads to linear time under the stated assumptions.

A branching recurrence such as:

```text
T(n) = 2T(n - 1) + O(1)
```

grows exponentially under the same simple model.

The exact recurrence must come from the actual algorithm.

---

## 22. Recursion Tree Reasoning

For branching algorithms, draw the call tree.

Example:

```text
                 solve(n)
              /            \
        solve(n-1)       solve(n-1)
        /      \          /      \
      ...      ...      ...      ...
```

This makes repeated work visible.

For recursive algorithms with overlapping subproblems, the tree can reveal why memoization may substantially reduce repeated computation.

For divide-and-conquer algorithms, a different recurrence structure often appears because the subproblem sizes shrink by a fraction rather than by one element.

---

## 23. Common Recursion Mistakes

### Mistake 1 — Missing base case

The recursion has no valid stopping condition.

### Mistake 2 — No progress toward the base case

The recursive call receives an equivalent or larger state without a deliberate reason.

### Mistake 3 — Wrong base-case result

The recursion terminates but returns the wrong mathematical or structural value.

### Mistake 4 — Misunderstanding return values

A recursive call may return a value that must be combined with the current state.

### Mistake 5 — Accidental repeated work

The same subproblem may be solved many times.

### Mistake 6 — Excessive recursion depth

A logically correct algorithm may still be unsuitable if the recursion depth can become too large for the runtime environment.

### Mistake 7 — Mutating shared state incorrectly

Backtracking requires state to be restored correctly after each branch.

---

## 24. Common Backtracking Mistakes

### Mistake 1 — Forgetting to undo a choice

The next branch inherits state from the previous branch.

### Mistake 2 — Recording incomplete solutions

Only record a solution when the problem's completion condition is satisfied.

### Mistake 3 — Missing pruning opportunities

If a partial state already violates a constraint, continuing the search wastes work.

### Mistake 4 — Duplicate solutions

The state transitions may produce the same logical solution through multiple paths.

### Mistake 5 — Incorrect start index

Combination-style problems often require a carefully defined starting position.

### Mistake 6 — Confusing permutations and combinations

Determine whether order matters before designing the state transitions.

---

## 25. A Repeatable Problem-Solving Process

Use this checklist for recursive problems.

### Step 1 — Define the function contract

Write exactly what one call means.

### Step 2 — Identify the smallest state

Determine the base case.

### Step 3 — Reduce the problem

Explain how the recursive case gets closer to the base case.

### Step 4 — Trace calls and returns

Use a tiny input.

### Step 5 — Analyze recursion depth

Determine maximum call-stack depth.

### Step 6 — Count repeated work

Look for overlapping subproblems.

### Step 7 — Consider iteration

Ask whether an iterative representation is clearer or safer for the expected input sizes.

For backtracking:

### Step 8 — Define the state

Example:

```text
path + remaining choices + constraints
```

### Step 9 — Define valid choices

Do not generate choices that are already impossible when the problem allows early rejection.

### Step 10 — Choose → recurse → undo

Verify that state is restored exactly after each branch.

---

## 26. Practice Progression

### Level 1 — Basic Recursion

1. Countdown.
2. Sum from `1` to `n`.
3. Factorial.
4. Reverse a sequence recursively.
5. Search a linked list recursively.

### Level 2 — Structural Recursion

6. Tree traversal.
7. Tree height.
8. Count nodes in a tree.
9. Search a tree.
10. Recursive binary search.

### Level 3 — Backtracking

11. Generate subsets.
12. Generate permutations.
13. Generate combinations.
14. Solve a constrained path problem.
15. Build a generic backtracking template.

### Level 4 — Optimization Reasoning

16. Identify repeated recursive states.
17. Add memoization.
18. Compare recursive and iterative implementations.
19. Derive a recurrence.
20. Estimate recursion depth and auxiliary space.

---

## 27. Mini Project — Configuration Combination Explorer

Build a small educational tool that explores valid combinations of configuration choices.

Example input:

```text
regions = ["dev", "staging", "prod"]
features = ["logging", "metrics"]
```

The tool can generate candidate configurations and reject combinations that violate explicitly defined rules.

### Required capabilities

- represent the current partial configuration
- choose the next option
- validate constraints
- recurse into the next decision
- undo the latest choice
- record complete valid configurations
- report how many candidates were explored
- explain which branches were pruned

### What to document

1. Function contract.
2. State representation.
3. Base case.
4. Choice generation.
5. Constraint checks.
6. Backtracking step.
7. Time complexity.
8. Auxiliary space.
9. Duplicate-handling rules.
10. Example trace.

This is a **repository learning project**. It is not a recommendation to use exhaustive in-memory search for unrestricted production configuration generation.

---

## 28. Interview Questions

### Conceptual

1. What makes a function recursive?
2. Why does recursion need a base case?
3. What is the call stack?
4. How is recursion different from iteration?
5. What is backtracking?

### Problem-solving

6. How would you reverse a linked list recursively?
7. How would you generate all subsets?
8. How would you generate permutations?
9. How would you prune an invalid search branch?
10. How would you prevent duplicate combinations?

### Complexity

11. How do you derive a recurrence from recursive code?
12. How do you calculate recursion depth?
13. Why can naive recursive solutions repeat work?
14. When can memoization improve a recursive solution?
15. Why can backtracking remain exponential even with pruning?

### Design

16. What should one recursive call represent?
17. What state must be restored after a backtracking branch?
18. When would you prefer iteration?
19. How would you test a recursive algorithm?
20. How would you explain the invariant of a backtracking solution?

---

## 29. Completion Checklist

You are ready to move on when you can:

- [ ] Define a recursive function contract.
- [ ] Identify a correct base case.
- [ ] Explain why each recursive call makes progress.
- [ ] Trace both recursive calls and returns.
- [ ] Estimate recursion depth.
- [ ] Write simple recursive algorithms.
- [ ] Explain recursive tree traversal.
- [ ] Recognize repeated subproblems.
- [ ] Explain memoization at a high level.
- [ ] Define a backtracking state.
- [ ] Generate subsets recursively.
- [ ] Generate permutations recursively.
- [ ] Prune invalid branches.
- [ ] Restore state after exploring a branch.
- [ ] Analyze time and auxiliary space.

---

## 30. Sources and Quality Notes

### Verified Python references

- Python Tutorial — More on Functions: https://docs.python.org/3/tutorial/controlflow.html#more-on-defining-functions
- Python `sys` documentation — recursion limit: https://docs.python.org/3/library/sys.html#sys.getrecursionlimit

### Repository design decisions

- Examples are intentionally small so the recursive state can be traced by hand.
- Backtracking examples emphasize state, invariants, and undo operations rather than memorized templates.
- Complexity statements are tied to the described algorithm and input model.
- Production recommendations are kept separate from DSA demonstrations.

### Accuracy policy

This chapter avoids treating recursion or backtracking as universally superior to iterative or non-search approaches. Actual suitability depends on input constraints, runtime behavior, output requirements, and the specific algorithm.

---

[← Previous: Stacks and Queues](../05%20Stacks%20and%20Queues/README.md) | [Back to Complete DSA](../README.md) | [Next: Sorting →](../07%20Sorting/README.md)
