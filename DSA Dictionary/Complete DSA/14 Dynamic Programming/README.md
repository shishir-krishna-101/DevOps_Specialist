# 14 — Dynamic Programming

Dynamic programming (DP) is a problem-solving technique for problems that can be expressed in terms of smaller subproblems. Instead of solving the same subproblem repeatedly, a DP solution stores results and reuses them.

DP is especially useful when a problem has:

- **Overlapping subproblems:** the same smaller problem appears multiple times.
- **Optimal substructure:** an optimal answer can be built from answers to appropriately defined smaller problems.

These properties help identify DP opportunities, but a correct solution still needs a precise state definition, recurrence, base cases, and evaluation order.

---

## Learning objectives

By the end of this chapter, you should be able to:

- Explain overlapping subproblems and optimal substructure.
- Distinguish memoization from tabulation.
- Define a DP state in terms of a smaller subproblem.
- Write a recurrence and identify its base cases.
- Implement Fibonacci, climbing stairs, and coin change.
- Understand 0/1 knapsack and longest common subsequence.
- Recognize one-dimensional, two-dimensional, and grid DP patterns.
- Optimize memory when previous states are sufficient.
- Analyze DP time and space complexity.
- Validate a DP solution against small brute-force examples.

## 1. What is dynamic programming?

Dynamic programming solves a problem by defining smaller states and reusing the answers to those states.

Consider the Fibonacci sequence:

```text
F(0) = 0
F(1) = 1
F(n) = F(n - 1) + F(n - 2), for n >= 2
```

A direct recursive implementation:

```python
def fibonacci_recursive(n):
    if n < 0:
        raise ValueError("n must be non-negative")
    if n <= 1:
        return n
    return fibonacci_recursive(n - 1) + fibonacci_recursive(n - 2)
```

This implementation recalculates values. For example, F(3) is needed by both F(5) and F(4) while calculating F(6). As the input grows, repeated work becomes substantial.

Dynamic programming stores a result once and reuses it.

### The five questions for a DP problem

Before writing code, answer:

1. **State:** What does one subproblem represent?
2. **Transition:** How can a state be calculated from smaller states?
3. **Base cases:** Which smallest states can be answered directly?
4. **Evaluation order:** In what order must states be computed?
5. **Final answer:** Which state or combination of states answers the original question?

If these are unclear, the implementation is likely to become confusing.

## 2. Memoization: top-down dynamic programming

Memoization starts with a recursive definition and caches results.

```python
from functools import lru_cache


def fibonacci_memoized(n):
    if n < 0:
        raise ValueError("n must be non-negative")

    @lru_cache(maxsize=None)
    def solve(value):
        if value <= 1:
            return value
        return solve(value - 1) + solve(value - 2)

    return solve(n)


print(fibonacci_memoized(10))
```

### How it works

- The recursive function requests the answer for a state.
- If the state has already been computed, the cached result is returned.
- Otherwise, the function calculates the result and stores it.
- Later calls reuse the stored value.

For Fibonacci, there are n + 1 distinct states from 0 through n. Each state is computed once, and each computation performs constant additional work.

- Time: O(n).
- Cache storage: O(n).
- Recursive call-stack depth: O(n).

Memoization can be a natural first implementation when the recurrence is easier to express recursively.

## 3. Tabulation: bottom-up dynamic programming

Tabulation starts from known base cases and fills a table in an order that ensures required smaller states are already available.

```python
def fibonacci_tabulated(n):
    if n < 0:
        raise ValueError("n must be non-negative")
    if n <= 1:
        return n

    dp = [0] * (n + 1)
    dp[1] = 1

    for value in range(2, n + 1):
        dp[value] = dp[value - 1] + dp[value - 2]

    return dp[n]


print(fibonacci_tabulated(10))
```

- Time: O(n).
- Table storage: O(n).
- Recursion-stack space: O(1), because this implementation is iterative.

### Memoization vs tabulation

| Aspect | Memoization | Tabulation |
|---|---|---|
| Direction | Top-down | Bottom-up |
| Structure | Usually recursive | Usually iterative |
| Computation | Calculates states when requested | Fills states in a planned order |
| Repeated work | Cached | Avoided through table construction |
| Stack use | May use recursion stack | Usually avoids recursive stack |
| Useful when | Only some reachable states may be needed | State order is straightforward |

Neither approach is universally better. Choose based on the recurrence, reachable states, readability, and resource constraints.

## 4. State design: the most important DP skill

A state is a precise description of a smaller problem.

For example, in a basic climbing-stairs problem:

- You can climb either one or two steps at a time.
- Let dp[i] represent the number of distinct ways to reach step i.
- To reach step i, the previous step was either i - 1 or i - 2.

Therefore:

```text
dp[i] = dp[i - 1] + dp[i - 2]
```

With dp[0] = 1 and dp[1] = 1, the answer for n steps is dp[n].

The definition dp[i] is more important than the variable name. If the state does not clearly describe what it represents, it is difficult to derive or verify the transition.

### State-design checklist

- What input dimensions affect the subproblem?
- Does the state include enough information to make future decisions?
- Does it include unnecessary information?
- Can the state be reached from smaller states?
- Does the final state answer the original question?

A state may be one-dimensional, two-dimensional, or have more dimensions. Each additional dimension can increase memory and computation substantially.

## 5. Climbing stairs

Suppose there are n steps and each move can climb one or two steps. Count the number of distinct ways to reach the top.

### State and transition

Let dp[i] be the number of ways to reach step i.

```text
dp[0] = 1
dp[1] = 1
dp[i] = dp[i - 1] + dp[i - 2]
```

The empty sequence of moves is one way to be at step zero. This base-case convention makes the recurrence work cleanly.

### Implementation

```python
def climb_stairs(n):
    if n < 0:
        raise ValueError("n must be non-negative")

    if n <= 1:
        return 1

    two_steps_back = 1
    one_step_back = 1

    for _ in range(2, n + 1):
        current = one_step_back + two_steps_back
        two_steps_back = one_step_back
        one_step_back = current

    return one_step_back


print(climb_stairs(5))
```

Only the previous two states are required, so the implementation keeps two variables instead of the full table.

- Time: O(n).
- Auxiliary space: O(1).

## 6. Minimum coin change

Given coin denominations and a target amount, find the minimum number of coins needed to make that amount. Assume an unlimited supply of each listed denomination.

This is the general minimum-coin formulation. Unlike some special coin systems, arbitrary denominations do not guarantee that a greedy largest-coin-first strategy is optimal.

### State definition

Let dp[x] be the minimum number of coins required to make amount x.

For each coin c that does not exceed x:

```text
dp[x] = min(dp[x], 1 + dp[x - c])
```

Use:

- dp[0] = 0.
- An initial unreachable value for positive amounts.
- The final answer dp[amount], unless it remains unreachable.

### Implementation

```python
def minimum_coins(coins, amount):
    if amount < 0:
        raise ValueError("amount must be non-negative")
    if any(coin <= 0 for coin in coins):
        raise ValueError("coin denominations must be positive")

    unreachable = amount + 1
    dp = [unreachable] * (amount + 1)
    dp[0] = 0

    for current_amount in range(1, amount + 1):
        for coin in coins:
            if coin <= current_amount:
                previous = dp[current_amount - coin]
                if previous != unreachable:
                    dp[current_amount] = min(
                        dp[current_amount],
                        previous + 1,
                    )

    if dp[amount] == unreachable:
        return None

    return dp[amount]


print(minimum_coins([1, 3, 4], 6))
```

For denominations [1, 3, 4] and amount 6, the minimum is 2 coins: 3 + 3. A largest-coin-first greedy choice selects 4 + 1 + 1, which uses 3 coins.

### Complexity

Let A be the target amount and k be the number of denominations.

- Time: O(A × k).
- Table storage: O(A).

This analysis assumes the amount is a non-negative integer and each denomination is a positive integer.

## 7. 0/1 knapsack

In the 0/1 knapsack problem:

- Each item has a weight and a value.
- The knapsack has a maximum capacity.
- Each item can be selected at most once.
- The objective is to maximize total value without exceeding capacity.

### State definition

Let dp[i][w] be the maximum value achievable using the first i items with capacity w.

For item i, there are two possibilities:

1. Do not take the item.
2. Take the item, if its weight fits.

For item weight wt and value val:

```text
If wt > w:
    dp[i][w] = dp[i - 1][w]
Otherwise:
    dp[i][w] = max(
        dp[i - 1][w],
        val + dp[i - 1][w - wt]
    )
```

The base row represents using zero items: every capacity has value zero.

### Implementation

```python
def knapsack_01(weights, values, capacity):
    if len(weights) != len(values):
        raise ValueError("weights and values must have equal length")
    if capacity < 0:
        raise ValueError("capacity must be non-negative")
    if any(weight < 0 for weight in weights):
        raise ValueError("weights must be non-negative")

    item_count = len(weights)
    dp = [[0] * (capacity + 1) for _ in range(item_count + 1)]

    for i in range(1, item_count + 1):
        weight = weights[i - 1]
        value = values[i - 1]

        for current_capacity in range(capacity + 1):
            dp[i][current_capacity] = dp[i - 1][current_capacity]

            if weight <= current_capacity:
                take_value = (
                    value
                    + dp[i - 1][current_capacity - weight]
                )
                dp[i][current_capacity] = max(
                    dp[i][current_capacity],
                    take_value,
                )

    return dp[item_count][capacity]


weights = [2, 3, 4]
values = [4, 5, 7]
print(knapsack_01(weights, values, 5))
```

### Complexity

For n items and capacity W:

- Time: O(n × W).
- Table storage: O(n × W).

This is called pseudo-polynomial time: the complexity depends on the numeric capacity W, not only on the number of bits needed to represent W.

The implementation assumes non-negative integer weights and capacity. If zero-weight items are permitted, the recurrence still handles them because each item is considered only from the previous row.

## 8. Space optimization in 0/1 knapsack

The two-dimensional table can be reduced to one dimension because each row only needs values from the previous row.

```python
def knapsack_01_optimized(weights, values, capacity):
    if len(weights) != len(values):
        raise ValueError("weights and values must have equal length")
    if capacity < 0:
        raise ValueError("capacity must be non-negative")
    if any(weight < 0 for weight in weights):
        raise ValueError("weights must be non-negative")

    dp = [0] * (capacity + 1)

    for weight, value in zip(weights, values):
        for current_capacity in range(capacity, weight - 1, -1):
            dp[current_capacity] = max(
                dp[current_capacity],
                value + dp[current_capacity - weight],
            )

    return dp[capacity]
```

The capacity loop runs backwards. This is essential for 0/1 knapsack: it prevents the current item from being reused multiple times during the same item iteration.

- Time: O(n × W).
- Auxiliary storage: O(W).

For the unbounded knapsack variant, where an item may be used repeatedly, the iteration direction and recurrence differ. Do not copy the 0/1 implementation into that problem without changing the state update correctly.

## 9. Longest common subsequence (LCS)

A subsequence preserves the relative order of elements but does not need to be contiguous.

Given two strings, the longest common subsequence problem asks for the maximum length of a sequence that appears in both strings in the same relative order.

Example:

- First string: ABCDGH
- Second string: AEDFHR
- One longest common subsequence is ADH.
- Its length is 3.

### State definition

Let dp[i][j] be the LCS length for:

- The first i characters of string A.
- The first j characters of string B.

Transition:

- If the current characters match, add one to the diagonal previous state.
- Otherwise, take the maximum of dropping one character from either prefix.

```text
If A[i - 1] == B[j - 1]:
    dp[i][j] = 1 + dp[i - 1][j - 1]
Otherwise:
    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
```

The first row and first column are zero because an empty string has an LCS of length zero with any string.

### Implementation

```python
def lcs_length(first, second):
    rows = len(first) + 1
    columns = len(second) + 1
    dp = [[0] * columns for _ in range(rows)]

    for i in range(1, rows):
        for j in range(1, columns):
            if first[i - 1] == second[j - 1]:
                dp[i][j] = 1 + dp[i - 1][j - 1]
            else:
                dp[i][j] = max(
                    dp[i - 1][j],
                    dp[i][j - 1],
                )

    return dp[-1][-1]


print(lcs_length("ABCDGH", "AEDFHR"))
```

### Complexity

For strings of lengths m and n:

- Time: O(m × n).
- Table storage: O(m × n).

This implementation returns the length, not the subsequence itself. Reconstructing an actual LCS requires tracking or walking back through the table.

## 10. Grid dynamic programming

Grid problems often define a state for each cell and calculate it from neighboring cells.

Suppose a robot starts at the top-left of a grid and can move only right or down. Count the number of paths to the bottom-right, assuming every cell is open.

Let dp[r][c] represent the number of ways to reach cell (r, c).

For cells not on the first row or first column:

```text
dp[r][c] = dp[r - 1][c] + dp[r][c - 1]
```

The first row and first column each have one path in an unobstructed rectangular grid.

### Implementation

```python
def count_grid_paths(rows, columns):
    if rows <= 0 or columns <= 0:
        return 0

    dp = [[0] * columns for _ in range(rows)]

    for row in range(rows):
        dp[row][0] = 1

    for column in range(columns):
        dp[0][column] = 1

    for row in range(1, rows):
        for column in range(1, columns):
            dp[row][column] = (
                dp[row - 1][column]
                + dp[row][column - 1]
            )

    return dp[-1][-1]


print(count_grid_paths(3, 3))
```

- Time: O(rows × columns).
- Space: O(rows × columns).

A one-dimensional table can reduce space to O(columns) for this particular transition. If obstacles are added, blocked cells must be assigned zero paths and the recurrence must respect the obstacle rules.

## 11. Common DP patterns

| Pattern | State often represents | Example |
|---|---|---|
| Linear / 1D DP | Answer for a prefix or position | Climbing stairs |
| Knapsack DP | Best answer under a capacity | 0/1 knapsack |
| Grid DP | Answer for reaching a cell | Grid paths |
| Sequence DP | Answer for prefixes or subsequences | LCS |
| Interval DP | Best answer for a range | Matrix-chain multiplication |
| Tree DP | Best answer for a subtree under a state | Tree selection problems |
| Bitmask DP | Answer for a subset of items | Small assignment problems |

These are families of state designs, not fixed recipes. Derive the recurrence from the exact problem.

## 12. How to reconstruct a solution

Some DP questions ask only for an optimum value. Others ask which items, steps, or sequence produce that value.

To reconstruct a solution:

1. Store enough information to identify which transition produced each state, or retain a table from which the choice can be recovered.
2. Start at the final state.
3. Follow the chosen predecessor or compare neighboring states to determine the previous decision.
4. Continue until reaching a base state.
5. Reverse the collected decisions if they were traced backwards.

For 0/1 knapsack, reconstruction can walk backwards through the two-dimensional table and identify whether each item was included. The one-dimensional optimized version does not retain the full history in the same straightforward form.

## 13. DP problem-solving workflow

Use this sequence before and during implementation:

1. **Understand the objective.** Is the task counting, minimizing, maximizing, or checking feasibility?
2. **Try a small example by hand.** Write down repeated decisions and subproblems.
3. **Look for overlap.** Are the same subproblems solved more than once by a natural recursive solution?
4. **Define the state.** Write one sentence explaining exactly what dp[...] means.
5. **Derive the transition.** Enumerate the possible final decisions and relate each to smaller states.
6. **Set base cases.** Check empty, zero, and smallest valid inputs.
7. **Choose evaluation order.** Ensure every dependency is computed before it is used.
8. **Implement a clear version.** Prefer a complete table initially if it makes the recurrence easy to inspect.
9. **Test edge cases.** Include impossible states, repeated values, and minimum dimensions.
10. **Optimize only when justified.** Reduce dimensions or memory after the state dependencies are understood.
11. **Analyze complexity.** Count states, work per state, and storage per state.
12. **Reconstruct if required.** A correct optimum value is not enough if the task asks for the actual choices.

## 14. Common DP mistakes

- Writing code before defining the state.
- Using a recurrence that does not match the problem's allowed choices.
- Forgetting base cases.
- Computing states in an order that uses values not yet calculated.
- Confusing a count of ways with a minimum cost or maximum value.
- Reusing an item accidentally in a 0/1 knapsack update.
- Treating an unreachable state as a valid numeric answer.
- Using an arbitrary sentinel without ensuring it cannot be confused with a valid result.
- Optimizing memory before understanding which previous states are needed.
- Reporting O(n) without counting the number of states and work per state.
- Returning the optimal value when the problem also requires the chosen items or sequence.

## 15. Testing and validation

For each DP implementation:

- Test the smallest valid input.
- Test zero capacity or zero target where meaningful.
- Test no items, no coins, or empty strings.
- Test impossible targets.
- Test repeated denominations or repeated values.
- Test cases where taking an option is worse than skipping it.
- Test cases where a locally attractive choice is not globally optimal.
- Compare against brute force for small inputs when practical.

A brute-force comparison can reveal implementation errors. It does not replace a correctness argument for the recurrence.

For optimized implementations, compare their outputs against the clear full-table implementation over many small test cases.

## 16. Practice progression

### Beginner

1. Implement Fibonacci using memoization.
2. Implement Fibonacci using tabulation.
3. Implement climbing stairs with a full table, then reduce it to two variables.
4. Implement minimum coin change and test the [1, 3, 4] counterexample.

### Intermediate

5. Implement 0/1 knapsack with a two-dimensional table.
6. Reduce 0/1 knapsack storage to one dimension and explain why the capacity loop runs backwards.
7. Implement LCS length.
8. Implement grid path counting.

### Challenge

9. Reconstruct one optimal set of items for 0/1 knapsack.
10. Reconstruct an actual longest common subsequence.
11. Solve a weighted interval scheduling problem using a DP state and recurrence.
12. Compare memoization and tabulation for a problem with unreachable states.
13. Design an interval DP or tree DP state for a small problem and justify its dependencies.

## 17. Mini project — Workload Optimization Planner

<badge color="info">Repository demonstration</badge>

Build a small program that chooses a set of jobs under a resource budget.

### Suggested version: 0/1 capacity planning

Each job has:

- A resource cost.
- A value or priority score.
- A rule that it can be selected at most once.

The program should:

- Accept a list of jobs and a total capacity.
- Use 0/1 knapsack DP to calculate the maximum achievable value.
- Display the maximum value.
- Extend the implementation to reconstruct which jobs were selected.
- Validate that the selected total cost does not exceed capacity.

### Suggested test cases

- No jobs.
- Zero capacity.
- One job that fits.
- One job that does not fit.
- Multiple jobs where a combination beats the highest-value individual job.
- Duplicate costs.
- Zero-cost jobs, if the project permits them.
- Invalid negative costs.

### Engineering extension

Compare the DP result with exhaustive subset enumeration for small job lists. This provides a useful correctness cross-check for the demonstration.

This project is a repository-designed exercise. Its job values and resource costs are teaching inputs, not a real production scheduling model.

## 18. Interview questions

1. What is dynamic programming?
2. What are overlapping subproblems?
3. What is optimal substructure?
4. How does memoization differ from tabulation?
5. How do you define a DP state?
6. How do you derive a transition?
7. Why are base cases important?
8. Why does Fibonacci benefit from caching?
9. Why does the 0/1 knapsack one-dimensional implementation iterate capacity backwards?
10. What is the difference between 0/1 and unbounded knapsack?
11. What does pseudo-polynomial complexity mean in knapsack?
12. How would you reconstruct the selected items from a DP table?
13. How do you count the time complexity of a DP solution?
14. When can memory be reduced from a two-dimensional table to one dimension?
15. How would you validate a DP implementation against brute force?

## 19. Completion checklist

- [ ] I can explain overlapping subproblems and optimal substructure.
- [ ] I can distinguish memoization from tabulation.
- [ ] I can define a state in one precise sentence.
- [ ] I can derive a recurrence from the available choices.
- [ ] I can identify correct base cases and evaluation order.
- [ ] I can implement Fibonacci using both DP approaches.
- [ ] I can solve minimum coin change.
- [ ] I understand the 0/1 knapsack recurrence.
- [ ] I understand why the one-dimensional 0/1 knapsack loop runs backwards.
- [ ] I can implement LCS length and grid path counting.
- [ ] I can calculate DP time and space complexity.
- [ ] I can reconstruct a solution when required.
- [ ] I can compare a DP implementation with brute force on small inputs.

## 20. Sources and quality notes

### Python implementation references

- [Python functools — caching and lru_cache](https://docs.python.org/3/library/functools.html#functools.lru_cache)
- [Python built-in range](https://docs.python.org/3/library/functions.html#func-range)

These official references document the Python features used in the examples. They are not proofs of the mathematical correctness of the recurrences.

### Content boundaries

- **Algorithmic concepts:** The chapter presents standard DP formulations and states the assumptions for each example.
- **Python implementation:** Code is written as instructional reference material; validate it with tests before adapting it to another problem.
- **Repository design decision:** The Workload Optimization Planner is a learning project designed for this repository.
- **Not implied:** A DP recurrence is not correct simply because it produces plausible sample outputs. Define and justify the state, transition, base cases, and evaluation order for the exact problem.

---

[← Previous: 13 — Divide and Conquer](../13%20Divide%20and%20Conquer/README.md) · [Complete DSA home](../README.md) · [Next: 15 — Advanced Graph Algorithms](../15%20Advanced%20Graph%20Algorithms/README.md)
