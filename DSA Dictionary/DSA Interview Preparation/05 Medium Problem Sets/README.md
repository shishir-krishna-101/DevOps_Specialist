# 05 — Medium Problem Sets

Medium problems test more than syntax. They require you to combine data structures, recognize the right invariant, handle constraints precisely, and explain why an optimization is valid.

> **Learning goal:** Solve a representative set of medium-level problems independently, then explain the pattern, correctness argument, complexity, and important edge cases.

This is a curated learning set, not an exhaustive list of interview questions and not a guarantee of interview performance. Work through the problems in order, then revisit each problem without looking at its solution.

---

## How to use this section

For every problem:

1. Restate the input and output contract.
2. Identify assumptions that make a pattern valid.
3. Trace a small example by hand.
4. Describe a straightforward approach before optimizing.
5. State the invariant that makes the optimized approach correct.
6. Implement the solution and test boundary cases.
7. Explain time and auxiliary-space complexity.

The solutions use Python's standard library. The code favors clarity over clever shortcuts.

## Problem map

| # | Problem | Main pattern | Main skill |
|---:|---|---|---|
| 1 | Longest substring without repeating characters | Sliding window + hash map | Maintain a valid window |
| 2 | Product of array except self | Prefix/suffix products | Avoid division |
| 3 | Subarray sum equals K | Prefix sum + frequency map | Handle negative values |
| 4 | Group anagrams | Hash-based grouping | Design a canonical key |
| 5 | Top K frequent elements | Frequency map + heap | Separate counting from selection |
| 6 | Merge overlapping intervals | Sorting + scan | Maintain a merged range |
| 7 | Search in rotated sorted array | Modified binary search | Identify the sorted half |
| 8 | Daily temperatures | Monotonic stack | Resolve earlier unanswered positions |
| 9 | Number of islands | Grid DFS | Traverse connected components |
| 10 | Course schedule | Topological sort | Detect dependency cycles |
| 11 | Coin change | Dynamic programming | Build optimal subproblem answers |
| 12 | Longest increasing subsequence | Binary search + tails | Maintain a compact DP frontier |

---

## 1. Longest substring without repeating characters

**Pattern:** Sliding window + last-seen index map

### Problem

Given a string, return the length of its longest contiguous substring that contains no repeated characters.

**Contract and assumptions**

- The input is a Python string and may be empty.
- Characters are compared exactly as Python string elements; no case folding or Unicode normalization is performed.
- Return an integer length, not the substring itself.

**Example**

~~~python
text = "abcabcbb"
# Output: 3; one longest valid substring is "abc".
~~~

### Hint

Keep the left boundary of a window and remember the most recent index for each character. When a repeated character appears inside the current window, move the left boundary just past its previous occurrence.

### Worked solution

~~~python
def longest_unique_substring_length(text):
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


assert longest_unique_substring_length("") == 0
assert longest_unique_substring_length("abcabcbb") == 3
assert longest_unique_substring_length("bbbbb") == 1
assert longest_unique_substring_length("pwwkew") == 3
assert longest_unique_substring_length("abcdef") == 6
~~~

### Why it works

After processing each position, the active window contains no repeated character. If the current character was last seen at an index inside the window, moving the left boundary to one position after that index removes the earlier copy. If the previous occurrence is before the current window, it does not affect validity.

Each character is processed once, and the left boundary only moves forward.

- **Time:** O(n), where n is the string length.
- **Auxiliary space:** O(u), where u is the number of distinct characters stored; at most O(n).

**Common mistake:** Assigning the left boundary to the previous index plus one without checking whether that occurrence is already outside the current window. That can move the boundary backward.

---

## 2. Product of array except self

**Pattern:** Prefix and suffix products

### Problem

Given a list of integers, return a new list where each position contains the product of every input value except the value at that position.

**Contract and assumptions**

- The input contains at least two integers.
- Values may be zero or negative.
- Do not use division.
- Do not mutate the input list.

**Example**

~~~python
values = [1, 2, 3, 4]
# Output: [24, 12, 8, 6]
~~~

### Hint

At each index, multiply the product of all values to its left by the product of all values to its right. Store left products in the output, then make a reverse pass using one running right product.

### Worked solution

~~~python
def product_except_self(values):
    if len(values) < 2:
        raise ValueError("values must contain at least two integers")

    result = [1] * len(values)

    prefix_product = 1
    for index, value in enumerate(values):
        result[index] = prefix_product
        prefix_product *= value

    suffix_product = 1
    for index in range(len(values) - 1, -1, -1):
        result[index] *= suffix_product
        suffix_product *= values[index]

    return result


assert product_except_self([1, 2, 3, 4]) == [24, 12, 8, 6]
assert product_except_self([-1, 1, 0, -3, 3]) == [0, 0, 9, 0, 0]
assert product_except_self([2, 3]) == [3, 2]
assert product_except_self([0, 0, 5]) == [0, 0, 0]
~~~

### Why it works

Before the reverse pass, result[i] contains the product of all elements strictly to the left of index i. During the reverse pass, suffix_product contains the product of all elements strictly to the right. Their product is therefore the product of every input element except values[i].

The multiplicative identity, 1, makes the same logic work at both ends and handles zero and negative values without special cases.

- **Time:** O(n).
- **Auxiliary space:** O(1) beyond the returned output list.

**Common mistake:** Multiplying the current value into the prefix or suffix before writing the result. That accidentally includes the excluded element.

---

## 3. Subarray sum equals K

**Pattern:** Prefix sum + frequency map

### Problem

Given a list of integers and an integer target, return the number of non-empty contiguous subarrays whose sum equals target.

**Contract and assumptions**

- The list may contain positive, zero, and negative integers.
- Count subarrays by their positions; equal-valued subarrays at different positions count separately.
- Return an integer count.

**Example**

~~~python
values = [1, 1, 1]
target = 2
# Output: 2
~~~

### Hint

If the current prefix sum is P, a previous prefix sum of P - target identifies a subarray ending at the current position whose sum is target. Store how many times each prefix sum has occurred.

### Worked solution

~~~python
def count_subarrays_with_sum(values, target):
    prefix_frequency = {0: 1}
    prefix_sum = 0
    count = 0

    for value in values:
        prefix_sum += value
        needed = prefix_sum - target

        count += prefix_frequency.get(needed, 0)
        prefix_frequency[prefix_sum] = (
            prefix_frequency.get(prefix_sum, 0) + 1
        )

    return count


assert count_subarrays_with_sum([1, 1, 1], 2) == 2
assert count_subarrays_with_sum([1, 2, 3], 3) == 2
assert count_subarrays_with_sum([1, -1, 0], 0) == 3
assert count_subarrays_with_sum([], 0) == 0
assert count_subarrays_with_sum([3], 3) == 1
~~~

### Why it works

Let P be the sum through the current index and Q be a prefix sum before the subarray begins. The subarray sum is P - Q. It equals target exactly when Q = P - target.

The frequency map counts every earlier prefix with that required sum, so adding its frequency counts all matching subarrays ending at the current position. The initial entry {0: 1} represents the empty prefix and allows a matching subarray to begin at index zero.

The current prefix is added only after checking for the required earlier prefix, ensuring the subarray is non-empty.

- **Time:** O(n) expected, assuming average constant-time dictionary operations.
- **Auxiliary space:** O(n) in the worst case.

**Important distinction:** A variable-size sliding window is not generally valid when negative values are allowed. Adding or removing a negative value can move the sum in either direction. Prefix sums plus a frequency map handle arbitrary integer signs.

---

## 4. Group anagrams

**Pattern:** Hash map + canonical key

### Problem

Given a list of lowercase English words, group words that are anagrams of one another. Return a list of groups. The order of groups and the order of words inside each group do not matter.

**Contract and assumptions**

- Every input item contains only characters a through z.
- Anagrams contain the same characters with the same frequencies.
- Empty strings are allowed.

**Example**

~~~python
words = ["eat", "tea", "tan", "ate", "nat", "bat"]
# One valid output:
# [["eat", "tea", "ate"], ["tan", "nat"], ["bat"]]
~~~

### Hint

Use the sorted characters of each word as a key. All anagrams produce the same sorted key.

### Worked solution

~~~python
from collections import defaultdict


def group_anagrams(words):
    groups = defaultdict(list)

    for word in words:
        key = "".join(sorted(word))
        groups[key].append(word)

    return list(groups.values())


def normalize_groups(groups):
    return sorted(sorted(group) for group in groups)


assert normalize_groups(group_anagrams(
    ["eat", "tea", "tan", "ate", "nat", "bat"]
)) == normalize_groups([
    ["eat", "tea", "ate"], ["tan", "nat"], ["bat"]
])
assert group_anagrams([""]) == [[""]]
assert normalize_groups(group_anagrams(["abc", "bca", "xyz"])) == [
    ["abc", "bca"], ["xyz"]
]
assert group_anagrams([]) == []
~~~

### Why it works

Sorting a word's characters creates a canonical representation: two words have the same sorted representation if and only if they contain the same characters with the same multiplicities. The dictionary therefore places all and only anagrams under the same key.

- **Time:** O(m × k log k), where m is the number of words and k is the maximum word length.
- **Auxiliary space:** O(mk) for keys and grouped output, excluding implementation details of sorting.

**Alternative:** Because the input is restricted to lowercase English letters, a 26-count frequency tuple can be used as a key. That takes O(k + 26) work per word. Do not use this fixed-size key without confirming the character-set constraint.

---

## 5. Top K frequent elements

**Pattern:** Frequency map + bounded min-heap

### Problem

Given a list of integers and an integer k, return the k most frequent distinct values.

**Contract and assumptions**

- 1 <= k <= the number of distinct values.
- Input values are integers.
- If frequencies tie, smaller numeric values come first. This tie rule makes the output deterministic.
- Return values ordered from highest frequency to lowest, applying the tie rule.

**Example**

~~~python
values = [1, 1, 1, 2, 2, 3]
k = 2
# Output: [1, 2]
~~~

### Hint

Count first. Keep a heap of at most k candidates. Make the heap root the least desirable retained candidate, so it can be evicted when a better candidate arrives.

### Worked solution

~~~python
from collections import Counter
import heapq


def top_k_frequent(values, k):
    frequencies = Counter(values)
    distinct_count = len(frequencies)

    if not 1 <= k <= distinct_count:
        raise ValueError("k must be between 1 and the distinct value count")

    # Higher frequency is better; for a tie, smaller value is better.
    # (frequency, -value) puts the least desirable candidate at heap[0].
    heap = []

    for value, frequency in frequencies.items():
        candidate = (frequency, -value)

        if len(heap) < k:
            heapq.heappush(heap, candidate)
        elif candidate > heap[0]:
            heapq.heapreplace(heap, candidate)

    selected = [
        (-negative_value, frequency)
        for frequency, negative_value in heap
    ]
    selected.sort(key=lambda item: (-item[1], item[0]))

    return [value for value, frequency in selected]


assert top_k_frequent([1, 1, 1, 2, 2, 3], 2) == [1, 2]
assert top_k_frequent([4, 4, 5, 5, 6], 2) == [4, 5]
assert top_k_frequent([-1, -1, 2, 2, 3], 2) == [-1, 2]
assert top_k_frequent([7], 1) == [7]
~~~

### Why it works

The frequency map reduces the input to one candidate per distinct value. The heap stores at most k candidates. Tuple ordering means a lower frequency is less desirable; when frequencies tie, a larger numeric value is less desirable because its negated value is smaller.

When the heap is full, a new candidate replaces the root only if it is more desirable. At the end, the retained candidates are sorted using the stated output order.

- **Time:** O(n + d log k + k log k), where n is input length and d is the number of distinct values. The Counter pass is O(n) expected.
- **Auxiliary space:** O(d + k), including the frequency map and heap.

**Common mistake:** Using a heap without defining tie behavior. If the problem does not specify tie ordering, clarify whether any valid ordering is accepted.

---

## 6. Merge overlapping intervals

**Pattern:** Sort + linear scan

### Problem

Given a list of intervals, merge all overlapping intervals and return the merged intervals in ascending start order.

**Contract and assumptions**

- Each interval is a pair [start, end] with start <= end.
- Intervals are closed: [a, b] and [b, c] overlap at b.
- The input may be empty.
- The function returns new interval lists and does not mutate the input.

**Example**

~~~python
intervals = [[1, 3], [2, 6], [8, 10], [15, 18]]
# Output: [[1, 6], [8, 10], [15, 18]]
~~~

### Hint

Sort by start. Keep the last merged interval. If the next interval begins at or before its end, extend that end; otherwise, start a new merged interval.

### Worked solution

~~~python
def merge_intervals(intervals):
    for interval in intervals:
        if len(interval) != 2 or interval[0] > interval[1]:
            raise ValueError("each interval must be [start, end] with start <= end")

    if not intervals:
        return []

    ordered = sorted(intervals, key=lambda interval: interval[0])
    merged = [[ordered[0][0], ordered[0][1]]]

    for start, end in ordered[1:]:
        if start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])

    return merged


assert merge_intervals([[1, 3], [2, 6], [8, 10], [15, 18]]) == [
    [1, 6], [8, 10], [15, 18]
]
assert merge_intervals([[1, 4], [4, 5]]) == [[1, 5]]
assert merge_intervals([[1, 2]]) == [[1, 2]]
assert merge_intervals([]) == []
assert merge_intervals([[5, 7], [1, 3], [2, 6]]) == [[1, 7]]
~~~

### Why it works

Sorting ensures that no later interval can start before the current interval. Therefore, only the last merged interval can overlap the next interval. If the next start is no greater than the last merged end, the union is a single interval ending at the larger end. Otherwise, the next interval is disjoint and can be appended.

- **Time:** O(n log n), dominated by sorting.
- **Auxiliary space:** O(n) for the sorted copy and output.

**Important convention:** For half-open intervals such as [start, end), sharing an endpoint does not necessarily mean overlap. The comparison must match the problem's interval convention.

---

## 7. Search in rotated sorted array

**Pattern:** Modified binary search

### Problem

A list of distinct integers was sorted in ascending order and then rotated at an unknown pivot. Given a target, return its index or -1 if it is absent.

**Contract and assumptions**

- All values are distinct.
- The list may be empty.
- The input is not modified.

**Example**

~~~python
values = [4, 5, 6, 7, 0, 1, 2]
target = 0
# Output: 4
~~~

### Hint

At every iteration, at least one side of the midpoint is sorted. Identify that side, then check whether the target lies within its inclusive bounds.

### Worked solution

~~~python
def search_rotated(values, target):
    left = 0
    right = len(values) - 1

    while left <= right:
        middle = left + (right - left) // 2

        if values[middle] == target:
            return middle

        if values[left] <= values[middle]:
            # Left half is sorted.
            if values[left] <= target < values[middle]:
                right = middle - 1
            else:
                left = middle + 1
        else:
            # Right half is sorted.
            if values[middle] < target <= values[right]:
                left = middle + 1
            else:
                right = middle - 1

    return -1


assert search_rotated([4, 5, 6, 7, 0, 1, 2], 0) == 4
assert search_rotated([4, 5, 6, 7, 0, 1, 2], 3) == -1
assert search_rotated([1], 0) == -1
assert search_rotated([1], 1) == 0
assert search_rotated([], 5) == -1
assert search_rotated([6, 7, 1, 2, 3, 4, 5], 7) == 1
~~~

### Why it works

Because the original sequence was sorted and contains distinct values, at least one of the two halves around the midpoint is sorted. If the target falls within that sorted half's value range, discard the other half. Otherwise, discard the sorted half. The remaining search interval always contains the target if it has not already been found.

- **Time:** O(log n).
- **Auxiliary space:** O(1).

**Constraint warning:** With duplicate values, deciding which half is sorted can become ambiguous when boundary and midpoint values are equal. This implementation deliberately requires distinct values.

---

## 8. Daily temperatures

**Pattern:** Monotonic decreasing stack

### Problem

Given a list of daily temperatures, return a list where each position contains the number of days until a warmer temperature. If no warmer future day exists, use zero.

**Contract and assumptions**

- Temperatures are integers.
- A warmer day means a strictly greater temperature.
- The input may be empty.

**Example**

~~~python
temperatures = [73, 74, 75, 71, 69, 72, 76, 73]
# Output: [1, 1, 4, 2, 1, 1, 0, 0]
~~~

### Hint

Store indices of days that have not yet found a warmer day. When a warmer temperature arrives, it resolves earlier indices from the top of the stack.

### Worked solution

~~~python
def daily_temperatures(temperatures):
    answer = [0] * len(temperatures)
    unresolved = []

    for today, temperature in enumerate(temperatures):
        while unresolved and temperatures[unresolved[-1]] < temperature:
            previous_day = unresolved.pop()
            answer[previous_day] = today - previous_day

        unresolved.append(today)

    return answer


assert daily_temperatures([73, 74, 75, 71, 69, 72, 76, 73]) == [
    1, 1, 4, 2, 1, 1, 0, 0
]
assert daily_temperatures([30, 40, 50, 60]) == [1, 1, 1, 0]
assert daily_temperatures([60, 50, 40]) == [0, 0, 0]
assert daily_temperatures([70, 70]) == [0, 0]
assert daily_temperatures([]) == []
~~~

### Why it works

The stack contains indices whose next warmer day has not yet appeared. Their temperatures are in non-increasing order from bottom to top. When today's temperature is strictly warmer than the top unresolved temperature, today is the first warmer day for that earlier index: any intervening days have already been processed and were not warmer. Pop and record the distance. Each index is pushed once and popped at most once.

- **Time:** O(n).
- **Auxiliary space:** O(n).

**Common mistake:** Popping on greater-than-or-equal. Equal temperatures are not warmer, so the comparison must be strict.

---

## 9. Number of islands

**Pattern:** Grid traversal with depth-first search

### Problem

Given a rectangular grid containing "1" for land and "0" for water, count the number of islands. Land is connected horizontally or vertically, not diagonally.

**Contract and assumptions**

- The grid is a list of equal-length strings.
- The grid may have zero rows or zero columns.
- The input is not modified.
- An island is a maximal four-directionally connected group of land cells.

**Example**

~~~python
grid = [
    "11000",
    "11000",
    "00100",
    "00011",
]
# Output: 3
~~~

### Hint

Scan every cell. When an unvisited land cell is found, increment the island count and traverse all land cells connected to it. Track visited coordinates separately so the original grid remains unchanged.

### Worked solution

~~~python
def count_islands(grid):
    if not grid:
        return 0

    columns = len(grid[0])
    if any(len(row) != columns for row in grid):
        raise ValueError("grid must be rectangular")

    rows = len(grid)
    visited = set()
    islands = 0
    directions = ((1, 0), (-1, 0), (0, 1), (0, -1))

    for row in range(rows):
        for column in range(columns):
            if grid[row][column] != "1" or (row, column) in visited:
                continue

            islands += 1
            stack = [(row, column)]
            visited.add((row, column))

            while stack:
                current_row, current_column = stack.pop()

                for row_step, column_step in directions:
                    next_row = current_row + row_step
                    next_column = current_column + column_step
                    neighbor = (next_row, next_column)

                    in_bounds = (
                        0 <= next_row < rows
                        and 0 <= next_column < columns
                    )

                    if (
                        in_bounds
                        and grid[next_row][next_column] == "1"
                        and neighbor not in visited
                    ):
                        visited.add(neighbor)
                        stack.append(neighbor)

    return islands


assert count_islands(["11000", "11000", "00100", "00011"]) == 3
assert count_islands(["111", "010", "111"]) == 1
assert count_islands(["000", "000"]) == 0
assert count_islands(["1"]) == 1
assert count_islands([]) == 0
assert count_islands(["", ""]) == 0
~~~

### Why it works

Each time the outer scan finds unvisited land, it starts a traversal of exactly that cell's connected component. The traversal marks every reachable land cell as visited. No cell can belong to two different components, so each island increments the count exactly once.

- **Time:** O(R × C), where R and C are grid dimensions.
- **Auxiliary space:** O(R × C) in the worst case for visited coordinates and the explicit stack.

**Common mistake:** Including diagonal neighbors when the problem defines only horizontal and vertical connectivity.

---

## 10. Course schedule

**Pattern:** Directed graph + Kahn's topological sort

### Problem

There are num_courses labeled from 0 through num_courses - 1. Each prerequisite pair [course, prerequisite] means the prerequisite must be completed before the course. Return True if all courses can be completed, otherwise False.

**Contract and assumptions**

- num_courses is a non-negative integer.
- Every prerequisite pair contains valid course identifiers.
- Duplicate prerequisite pairs are allowed and must not change the answer.
- A cycle makes completion impossible.

**Example**

~~~python
num_courses = 2
prerequisites = [[1, 0]]
# Output: True
~~~

~~~python
num_courses = 2
prerequisites = [[1, 0], [0, 1]]
# Output: False
~~~

### Hint

Create a directed edge from prerequisite to course. Count each course's incoming prerequisites. Repeatedly process courses with zero incoming edges. If you process every course, the graph is acyclic.

### Worked solution

~~~python
from collections import deque


def can_finish_courses(num_courses, prerequisites):
    if num_courses < 0:
        raise ValueError("num_courses cannot be negative")

    adjacency = [set() for _ in range(num_courses)]
    indegree = [0] * num_courses

    for course, prerequisite in prerequisites:
        if not (
            0 <= course < num_courses
            and 0 <= prerequisite < num_courses
        ):
            raise ValueError("course identifier is out of range")

        if course not in adjacency[prerequisite]:
            adjacency[prerequisite].add(course)
            indegree[course] += 1

    ready = deque(
        course for course in range(num_courses)
        if indegree[course] == 0
    )
    completed = 0

    while ready:
        prerequisite = ready.popleft()
        completed += 1

        for course in adjacency[prerequisite]:
            indegree[course] -= 1
            if indegree[course] == 0:
                ready.append(course)

    return completed == num_courses


assert can_finish_courses(2, [[1, 0]]) is True
assert can_finish_courses(2, [[1, 0], [0, 1]]) is False
assert can_finish_courses(3, [[1, 0], [2, 1]]) is True
assert can_finish_courses(1, []) is True
assert can_finish_courses(0, []) is True
assert can_finish_courses(2, [[1, 0], [1, 0]]) is True
~~~

### Why it works

A course with in-degree zero has no remaining prerequisite, so it can be completed and removed from the dependency graph. Removing it decreases the in-degree of each dependent course. If all vertices are removed, the graph has a topological ordering and contains no directed cycle. If processing stops while vertices remain, each remaining vertex is blocked by dependencies within the remaining graph, which implies a directed cycle.

Using a set for each adjacency list prevents duplicate edges from increasing in-degree more than once.

- **Time:** O(V + E) expected, where V is the number of courses and E is the number of distinct prerequisite edges.
- **Auxiliary space:** O(V + E).

**Common mistake:** Reversing the edge direction. For [course, prerequisite], the graph edge must go from prerequisite to course.

---

## 11. Coin change

**Pattern:** Bottom-up dynamic programming

### Problem

Given a list of positive coin denominations and a non-negative amount, return the minimum number of coins needed to make that amount. Return -1 if it cannot be formed.

**Contract and assumptions**

- Every denomination is a positive integer.
- amount is a non-negative integer.
- Each denomination can be used an unlimited number of times.
- Duplicate denominations are permitted but do not change the answer.

**Example**

~~~python
coins = [1, 2, 5]
amount = 11
# Output: 3, because 5 + 5 + 1 uses three coins.
~~~

### Hint

Let dp[x] represent the minimum number of coins needed to form amount x. For each amount, try every denomination that does not exceed it and extend a previously solved smaller amount.

### Worked solution

~~~python
def minimum_coins(coins, amount):
    if amount < 0:
        raise ValueError("amount cannot be negative")
    if any(coin <= 0 for coin in coins):
        raise ValueError("coin denominations must be positive integers")

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
                        previous + 1
                    )

    return -1 if dp[amount] == unreachable else dp[amount]


assert minimum_coins([1, 2, 5], 11) == 3
assert minimum_coins([2], 3) == -1
assert minimum_coins([1], 0) == 0
assert minimum_coins([2, 3, 7], 12) == 3
assert minimum_coins([2, 5], 1) == -1
~~~

### Why it works

The base case is dp[0] = 0 because no coins are needed to make zero. For a positive amount x, any valid final coin must have denomination c no greater than x. If dp[x - c] is reachable, adding that coin gives a candidate solution with dp[x - c] + 1 coins. Taking the minimum over all denominations considers every possible final coin.

Because every denomination is positive, x - c is smaller than x and has already been computed in the ascending loop.

- **Time:** O(A × C), where A is the amount and C is the number of denominations.
- **Auxiliary space:** O(A).

**Common mistake:** Assuming a greedy strategy that repeatedly chooses the largest coin is always optimal. It is not valid for arbitrary denomination systems; dynamic programming checks all possible final denominations.

---

## 12. Longest increasing subsequence

**Pattern:** Dynamic programming frontier + binary search

### Problem

Given a list of integers, return the length of its longest strictly increasing subsequence. A subsequence preserves relative order but does not need to be contiguous.

**Contract and assumptions**

- Values may be positive, zero, or negative.
- The input may be empty.
- The subsequence must be strictly increasing; equal values cannot extend it.

**Example**

~~~python
values = [10, 9, 2, 5, 3, 7, 101, 18]
# Output: 4; one answer is [2, 3, 7, 18].
~~~

### Hint

Maintain tails[i], the smallest possible ending value of an increasing subsequence of length i + 1 found so far. For each value, replace the first tail that is greater than or equal to it, or append if all tails are smaller.

### Worked solution

~~~python
from bisect import bisect_left


def longest_increasing_subsequence_length(values):
    tails = []

    for value in values:
        position = bisect_left(tails, value)

        if position == len(tails):
            tails.append(value)
        else:
            tails[position] = value

    return len(tails)


assert longest_increasing_subsequence_length(
    [10, 9, 2, 5, 3, 7, 101, 18]
) == 4
assert longest_increasing_subsequence_length([]) == 0
assert longest_increasing_subsequence_length([7, 7, 7]) == 1
assert longest_increasing_subsequence_length([1, 2, 3, 4]) == 4
assert longest_increasing_subsequence_length([4, 3, 2, 1]) == 1
assert longest_increasing_subsequence_length([0, 8, 4, 12, 2]) == 3
~~~

### Why it works

For every achievable subsequence length, keeping the smallest known tail is sufficient: a smaller tail leaves at least as many future values available to extend that subsequence. Processing a new value replaces the first tail that is greater than or equal to it, or extends the list if the value is larger than every current tail. The number of tails equals the length of the longest increasing subsequence.

The tails list is a compact summary; it is not necessarily itself a subsequence of the input. This implementation returns the length only.

- **Time:** O(n log L), where L is the length of the longest increasing subsequence; bounded by O(n log n).
- **Auxiliary space:** O(L).

**Strict versus non-decreasing:** bisect_left finds the first tail greater than or equal to the current value, which enforces a strictly increasing subsequence. A non-decreasing variant requires a different boundary choice.

---

## Review: choose the pattern from the constraints

| If the problem requires... | Consider... | Verify this first |
|---|---|---|
| A longest valid contiguous substring | Sliding window | Can the left boundary move monotonically? |
| A contiguous sum with negative values | Prefix sums + map | Are you counting earlier prefix sums? |
| Repeated grouping by equivalent content | Canonical hash key | Does the key preserve exactly the required equivalence? |
| A small top-K result | Bounded heap | What is the tie-breaking rule? |
| Overlap consolidation | Sort + scan | Are intervals closed or half-open? |
| Search in rotated ordered data | Modified binary search | Are duplicates allowed? |
| Next greater future value | Monotonic stack | Is the comparison strict? |
| Connected grid regions | DFS or BFS | Which directions define connectivity? |
| Dependency feasibility | Topological sort | Which way do prerequisite edges point? |
| Minimum number of choices | Dynamic programming | What are the base cases and transitions? |
| Longest increasing subsequence length | Binary-search tails | Strictly increasing or non-decreasing? |

A pattern is a candidate, not a keyword-based guarantee. Confirm that the input properties support the algorithm before using it.

## Testing checklist

Before considering a solution complete, test the cases that are relevant to its contract:

- Empty input and the smallest permitted input.
- One element and repeated elements.
- Negative values and zero where allowed.
- Already sorted and reverse-sorted inputs.
- No valid answer and multiple valid answers.
- Boundary equality: less-than versus less-than-or-equal.
- Duplicate edges, intervals, or values where allowed.
- Whether the function mutates its input.
- Whether output ordering is specified or intentionally unspecified.

For larger inputs, add randomized or property-based comparisons against a simple brute-force implementation where practical. A brute-force implementation is useful as a test oracle on small inputs, even when it is too slow for production-sized inputs.

## Suggested practice schedule

| Session | Work |
|---|---|
| 1 | Problems 1–3: windows and prefix sums |
| 2 | Problems 4–5: grouping, counting, and heaps |
| 3 | Problems 6–8: intervals, binary search, and monotonic stacks |
| 4 | Problems 9–10: graph traversal and dependencies |
| 5 | Problems 11–12: dynamic programming |
| 6 | Re-solve four problems without notes; explain invariants aloud |
| 7 | Take a timed mixed set and review mistakes |

Do not measure progress only by the number of completed problems. Track whether you can derive the approach, justify it, implement it, and test it without relying on the worked solution.

## Completion checklist

- [ ] I can explain the input contract and assumptions for every problem.
- [ ] I can identify the pattern from the constraints rather than from a memorized title.
- [ ] I can state the key invariant or DP transition.
- [ ] I can implement each solution without copying.
- [ ] I can explain time and auxiliary-space complexity accurately.
- [ ] I can identify at least two edge cases for each problem.
- [ ] I can explain when the chosen pattern would not be valid.
- [ ] I can solve a mixed selection under a time limit.

## Navigation

- [DSA Interview Preparation — Main Guide](../README.md)
- [01 — Interview Roadmap](../01%20Interview%20Roadmap/README.md)
- [02 — Python Coding Interview](../02%20Python%20Coding%20Interview/README.md)
- [03 — Pattern Recognition](../03%20Pattern%20Recognition/README.md)
- [04 — Easy Problem Sets](../04%20Easy%20Problem%20Sets/README.md)
- [Complete DSA — Interview Patterns](../../Complete%20DSA/17%20Interview%20Patterns/README.md)
- [Complete DSA — DSA Practice Problems](../../Complete%20DSA/18%20DSA%20Practice%20Problems/README.md)

## Official Python references

These official references support the Python APIs used in the examples:

- [Built-in Types — Python documentation](https://docs.python.org/3/library/stdtypes.html)
- [collections — Container datatypes](https://docs.python.org/3/library/collections.html)
- [heapq — Heap queue algorithm](https://docs.python.org/3/library/heapq.html)
- [bisect — Array bisection algorithm](https://docs.python.org/3/library/bisect.html)
- [unittest — Unit testing framework](https://docs.python.org/3/library/unittest.html)

The algorithms and problem contracts in this file are educational implementations and repository design choices. The Python documentation verifies the referenced language and standard-library APIs; it does not prescribe these interview problems or guarantee a particular interview outcome.
