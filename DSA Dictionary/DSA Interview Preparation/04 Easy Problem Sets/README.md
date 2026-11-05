# 04 — Easy Problem Sets

This section provides guided easy-level coding interview exercises. Each problem has a clear contract, a worked Python solution, reasoning, complexity analysis, and tests or edge cases.

The problems are original learning exercises for this repository. The easy label is a **curriculum classification**, not a claim that every employer or interview platform uses the same difficulty scale.

> **Learning goal:** Solve foundational problems independently, explain why the solution works, write clean Python, and test boundaries before moving to medium-level problems.

---

## 1. How to use these problem sets

For each problem:

1. Rewrite the input/output contract in your own words.
2. Work through the example without code.
3. Attempt a solution before reading the worked answer.
4. Identify the data structure or pattern that removes unnecessary work.
5. State the invariant or reasoning that makes the solution correct.
6. Estimate time and auxiliary-space complexity.
7. Run the edge-case tests and add one of your own.
8. Reimplement the solution from memory later.

A problem is learned when you can explain the approach, implement it, test it, and recognize when its assumptions do not apply.

| Set | Focus | Problems |
|---|---|---|
| A — Fundamentals | Traversal, counters, strings, sets | 1–4 |
| B — Arrays and lookup | Hash maps, in-place updates, scanning | 5–7 |
| C — Core structures | Binary search, stacks, merging | 8–10 |
| D — Interview foundations | Single-pass optimization, linked-list pointers | 11–12 |

---

## 2. Set A — Fundamentals

### Problem 1 — Count target occurrences

<badge color="success">Easy</badge> <badge>Array traversal</badge>

**Task:** Given a sequence of integers and a target integer, return the number of elements equal to the target.

**Contract:** The sequence may be empty. Negative values and duplicates are allowed. Do not modify the input.

**Example:** [4, 2, 4, 4, 7], target 4 → 3

**Worked solution**

~~~python
def count_occurrences(values, target):
    count = 0

    for value in values:
        if value == target:
            count += 1

    return count
~~~

**Reasoning:** Every element could be the target, so examine each element once. The counter records the number of matches.

**Complexity:** Time O(n); auxiliary space O(1).

**Tests**

~~~python
assert count_occurrences([4, 2, 4, 4, 7], 4) == 3
assert count_occurrences([], 4) == 0
assert count_occurrences([-1, -1, 0], -1) == 2
assert count_occurrences([1, 2, 3], 9) == 0
~~~

**Common mistake:** Returning a boolean after finding a match. The task asks for a count, not membership.

---

### Problem 2 — Second-largest distinct value

<badge color="success">Easy</badge> <badge>Single pass</badge>

**Task:** Return the second-largest distinct integer. If fewer than two distinct values exist, return None.

**Contract:** Empty input and duplicates are allowed. Do not sort or modify the input. Values are integers, so None can represent an uninitialized result.

**Example:** [8, 3, 8, 5, 5] → 5

**Worked solution**

~~~python
def second_largest_distinct(values):
    largest = None
    second = None

    for value in values:
        if largest is None or value > largest:
            second = largest
            largest = value
        elif value != largest and (second is None or value > second):
            second = value

    return second
~~~

**Reasoning:** Maintain the largest and second-largest distinct values seen so far. A new maximum moves the old maximum into the second position. A duplicate maximum is ignored.

**Complexity:** Time O(n); auxiliary space O(1).

**Tests**

~~~python
assert second_largest_distinct([8, 3, 8, 5, 5]) == 5
assert second_largest_distinct([10, 10, 10]) is None
assert second_largest_distinct([1, 2]) == 1
assert second_largest_distinct([-5, -2, -9]) == -5
assert second_largest_distinct([]) is None
~~~

**Common mistake:** Treating the second occurrence of the maximum as the second-largest value. The requirement is distinctness.

---

### Problem 3 — Reverse the order of words

<badge color="success">Easy</badge> <badge>Strings</badge>

**Task:** Return the words in a sentence in reverse order, separated by one space. Do not reverse characters inside a word.

**Contract:** Runs of whitespace separate words. Leading/trailing whitespace is ignored. An input with no words returns an empty string.

**Example:** "  learn   data structures " → "structures data learn"

**Worked solution**

~~~python
def reverse_words(sentence):
    words = sentence.split()
    words.reverse()
    return " ".join(words)
~~~

**Reasoning:** Calling split without an explicit separator splits on runs of whitespace and does not create empty tokens for leading, trailing, or repeated whitespace. Reverse the resulting words and join with one space.

**Complexity:** Time O(c); auxiliary space O(c), where c is the number of input characters.

**Tests**

~~~python
assert reverse_words("learn data structures") == "structures data learn"
assert reverse_words("  learn   data  ") == "data learn"
assert reverse_words("") == ""
assert reverse_words("   \t\n ") == ""
assert reverse_words("single") == "single"
~~~

**Common mistake:** Reversing the entire string, which also reverses characters inside each word.

---

### Problem 4 — Check whether two strings are anagrams

<badge color="success">Easy</badge> <badge>Frequency counting</badge>

**Task:** Return True when two strings contain exactly the same characters with the same multiplicities; otherwise return False.

**Contract:** Comparison is case-sensitive. Spaces and punctuation are characters. No Unicode normalization is performed.

**Examples:** "listen", "silent" → True; "hello", "world" → False; "aab", "aba" → True.

**Worked solution**

~~~python
from collections import Counter


def are_anagrams(first, second):
    return Counter(first) == Counter(second)
~~~

**Reasoning:** Anagrams have the same frequency for every character. Counter creates a mapping from each character to its count. Equal mappings mean equal character multiplicities.

**Complexity:** Time O(a+b) expected; auxiliary space O(u), where a and b are string lengths and u is the number of distinct characters. Expected hash-table costs assume average constant-time operations.

**Tests**

~~~python
assert are_anagrams("listen", "silent") is True
assert are_anagrams("aab", "aba") is True
assert are_anagrams("hello", "world") is False
assert are_anagrams("", "") is True
assert are_anagrams("A", "a") is False
assert are_anagrams("a b", "ab ") is True
~~~

**Common mistake:** Comparing only sets of characters. A set discards multiplicity and cannot distinguish "aab" from "abb".

---

## 3. Set B — Arrays and lookup

### Problem 5 — Two Sum: return indices

<badge color="success">Easy</badge> <badge>Hash map</badge>

**Task:** Return indices of two different elements whose values add to the target. Return None if no pair exists.

**Contract:** Input is not guaranteed sorted. Each position may be used once. If multiple pairs exist, any valid pair is acceptable. Returned indices are in discovery order.

**Example:** values [6, 1, 8, 3], target 9 → [0, 2]

**Worked solution**

~~~python
def two_sum_indices(values, target):
    seen = {}

    for index, value in enumerate(values):
        needed = target - value

        if needed in seen:
            return [seen[needed], index]

        seen[value] = index

    return None
~~~

**Reasoning:** For each value, calculate the complement needed to reach the target. The dictionary stores earlier values and their indices. Check before inserting the current value so the same position cannot be used twice.

**Complexity:** Expected time O(n); auxiliary space O(n). The expected time assumes average constant-time dictionary lookup and insertion.

**Tests**

~~~python
assert two_sum_indices([6, 1, 8, 3], 9) == [0, 2]
assert two_sum_indices([3, 3], 6) == [0, 1]
assert two_sum_indices([1, 2, 3], 7) is None
assert two_sum_indices([], 0) is None
assert two_sum_indices([-2, 5, 7], 3) == [0, 1]
~~~

**Common mistakes:** Sorting and returning indices from the sorted list; inserting before checking the complement; assuming values are positive.

---

### Problem 6 — Move zeroes to the end

<badge color="success">Easy</badge> <badge>In-place array update</badge>

**Task:** Move every zero to the end of a list while preserving the relative order of all non-zero values.

**Contract:** Modify the list in place. The function returns None.

**Example:** [0, 1, 0, 3, 12] → [1, 3, 12, 0, 0]

**Worked solution**

~~~python
def move_zeroes(values):
    write = 0

    for value in values:
        if value != 0:
            values[write] = value
            write += 1

    while write < len(values):
        values[write] = 0
        write += 1
~~~

**Reasoning:** write identifies the next position for a non-zero value. The scan reads values left to right and writes non-zero values in their original relative order. Then fill the remaining positions with zeroes. The write position never moves ahead of the scan position, so an unread value is not overwritten.

**Complexity:** Time O(n); auxiliary space O(1). Input mutation: yes.

**Tests**

~~~python
values = [0, 1, 0, 3, 12]
assert move_zeroes(values) is None
assert values == [1, 3, 12, 0, 0]

values = [0, 0]
move_zeroes(values)
assert values == [0, 0]

values = [1, 2, 3]
move_zeroes(values)
assert values == [1, 2, 3]

values = []
move_zeroes(values)
assert values == []
~~~

**Common mistake:** Repeatedly removing zeroes from the middle of a list. Later elements shift after each removal, which can make the solution quadratic.

---

### Problem 7 — First unique character index

<badge color="success">Easy</badge> <badge>Frequency map</badge>

**Task:** Return the index of the first character that occurs exactly once. Return -1 if no such character exists.

**Contract:** Comparison is case-sensitive. Empty input returns -1. Return an index, not the character.

**Example:** "swiss" → 1, because w is the first character that occurs once.

**Worked solution**

~~~python
from collections import Counter


def first_unique_index(text):
    counts = Counter(text)

    for index, character in enumerate(text):
        if counts[character] == 1:
            return index

    return -1
~~~

**Reasoning:** The first pass builds frequencies. The second pass scans the original string in order and returns the first index whose frequency is one. An early character cannot be known to be unique until the whole string has been counted.

**Complexity:** Time O(n) expected; auxiliary space O(u), where u is the number of distinct characters.

**Tests**

~~~python
assert first_unique_index("swiss") == 1
assert first_unique_index("aabb") == -1
assert first_unique_index("z") == 0
assert first_unique_index("") == -1
assert first_unique_index("aAbA") == 1
~~~

**Common mistake:** Returning the first character that differs from its immediate neighbor. Uniqueness is determined across the entire string.

---

## 4. Set C — Core data structures

### Problem 8 — Binary search

<badge color="success">Easy</badge> <badge>Binary search</badge>

**Task:** Given a list sorted in non-decreasing order and a target, return an index containing the target. Return -1 if absent.

**Contract:** Input is sorted ascending. Duplicates may exist. Any matching index is acceptable. Do not modify the input.

**Example:** [-4, 0, 3, 8, 12], target 8 → 3

**Worked solution**

~~~python
def binary_search(values, target):
    left = 0
    right = len(values) - 1

    while left <= right:
        middle = left + (right - left) // 2

        if values[middle] == target:
            return middle
        if values[middle] < target:
            left = middle + 1
        else:
            right = middle - 1

    return -1
~~~

**Reasoning:** The search interval contains every position that could still hold the target. If the middle value is too small, sorted order proves the middle and everything to its left can be discarded. If it is too large, discard the middle and everything to its right. Each iteration roughly halves the interval.

**Complexity:** Time O(log n); auxiliary space O(1).

**Tests**

~~~python
assert binary_search([-4, 0, 3, 8, 12], 8) == 3
assert binary_search([], 8) == -1
assert binary_search([5], 5) == 0
assert binary_search([1, 2, 2, 2, 4], 2) in (1, 2, 3)
assert binary_search([1, 2, 3], 9) == -1
~~~

**Common mistakes:** Applying binary search to unsorted input; using inconsistent interval boundaries; updating a bound without ensuring the interval shrinks.

---

### Problem 9 — Valid parentheses

<badge color="success">Easy</badge> <badge>Stack</badge>

**Task:** Determine whether a string containing only (), [], and {} is correctly balanced and nested.

**Contract:** Every opening delimiter must have a matching closing delimiter. Closing order must be properly nested. Empty input is valid.

**Examples:** "([]{})" → True; "([)]" → False; "(((" → False.

**Worked solution**

~~~python
def is_valid_parentheses(text):
    matching_open = {
        ")": "(",
        "]": "[",
        "}": "{",
    }
    stack = []

    for character in text:
        if character in "([{":
            stack.append(character)
        else:
            if not stack or stack[-1] != matching_open[character]:
                return False
            stack.pop()

    return not stack
~~~

**Reasoning:** The most recently opened delimiter must be the next one closed. A stack models this last-in, first-out rule. A closing delimiter is invalid if the stack is empty or its top contains a different opening type. At the end, the stack must be empty.

The input contract restricts characters to the six delimiter symbols. If other characters were allowed, their handling would need to be specified.

**Complexity:** Time O(n); auxiliary space O(n) in the worst case.

**Tests**

~~~python
assert is_valid_parentheses("([]{})") is True
assert is_valid_parentheses("([)]") is False
assert is_valid_parentheses("") is True
assert is_valid_parentheses("(((") is False
assert is_valid_parentheses(")") is False
assert is_valid_parentheses("{[]}") is True
~~~

**Common mistake:** Counting opening and closing delimiters without checking their order. Equal counts do not guarantee correct nesting.

---

### Problem 10 — Merge two sorted lists

<badge color="success">Easy</badge> <badge>Two pointers</badge>

**Task:** Given two lists sorted in non-decreasing order, return a new sorted list containing all values from both lists.

**Contract:** Duplicates are preserved. Do not modify either input. Either input may be empty.

**Example:** [1, 3, 7] and [2, 3, 8] → [1, 2, 3, 3, 7, 8]

**Worked solution**

~~~python
def merge_sorted_lists(first, second):
    i = 0
    j = 0
    merged = []

    while i < len(first) and j < len(second):
        if first[i] <= second[j]:
            merged.append(first[i])
            i += 1
        else:
            merged.append(second[j])
            j += 1

    merged.extend(first[i:])
    merged.extend(second[j:])

    return merged
~~~

**Reasoning:** The smaller of the two unprocessed values must be the next value in the merged result. The pointers identify the first unprocessed value in each input. When one input is exhausted, the remaining suffix of the other input is already sorted.

**Complexity:** Time O(m+n); output space O(m+n), where m and n are the input lengths.

**Tests**

~~~python
assert merge_sorted_lists([1, 3, 7], [2, 3, 8]) == [1, 2, 3, 3, 7, 8]
assert merge_sorted_lists([], [1, 2]) == [1, 2]
assert merge_sorted_lists([1, 2], []) == [1, 2]
assert merge_sorted_lists([], []) == []
assert merge_sorted_lists([2, 2], [2, 2]) == [2, 2, 2, 2]
~~~

**Common mistake:** Sorting the concatenation. That works, but ignores that both inputs are already sorted and usually costs more time.

---

## 5. Set D — Interview foundations

### Problem 11 — Best time to buy and sell once

<badge color="success">Easy</badge> <badge>Single-pass optimization</badge>

**Task:** Given daily stock prices, return the maximum profit possible from one buy followed by one later sell. If no positive profit is possible, return zero.

**Contract:** Prices are non-negative numbers. Buy must happen before sell. At most one transaction is allowed. Fewer than two prices returns zero.

**Example:** [7, 1, 5, 3, 6, 4] → 5, by buying at 1 and selling later at 6.

**Worked solution**

~~~python
def max_profit_once(prices):
    minimum_price = float("inf")
    best_profit = 0

    for price in prices:
        minimum_price = min(minimum_price, price)
        best_profit = max(best_profit, price - minimum_price)

    return best_profit
~~~

**Reasoning:** As the scan moves forward, minimum_price is the lowest price seen so far. Selling at the current price gives the best profit available with this sell day. Updating best_profit retains the best valid transaction found so far. The buy day cannot occur after the sell day.

**Complexity:** Time O(n); auxiliary space O(1).

**Tests**

~~~python
assert max_profit_once([7, 1, 5, 3, 6, 4]) == 5
assert max_profit_once([7, 6, 4, 3, 1]) == 0
assert max_profit_once([]) == 0
assert max_profit_once([5]) == 0
assert max_profit_once([2, 4]) == 2
~~~

**Common mistake:** Finding the global minimum and global maximum independently. The maximum may occur before the minimum, violating buy-before-sell order.

---

### Problem 12 — Find the middle of a linked list

<badge color="success">Easy</badge> <badge>Fast and slow pointers</badge>

**Task:** Return the middle node of a singly linked list. For an even number of nodes, return the second of the two middle nodes.

**Contract:** The list may be empty. Node values are integers. Return the node itself, not only its value. Do not modify the list.

**Examples:** 1 → 2 → 3 → 4 → 5 returns node 3. 1 → 2 → 3 → 4 → 5 → 6 returns node 4.

**Worked solution**

~~~python
class ListNode:
    def __init__(self, value, next_node=None):
        self.value = value
        self.next = next_node


def middle_node(head):
    slow = head
    fast = head

    while fast is not None and fast.next is not None:
        slow = slow.next
        fast = fast.next.next

    return slow
~~~

**Reasoning:** The slow pointer advances one node per iteration; the fast pointer advances two. When the fast pointer reaches the end, slow has covered half the list. This loop condition makes slow land on the second middle node for even-length lists.

**Complexity:** Time O(n); auxiliary space O(1).

**Tests**

~~~python
def build_linked_list(values):
    head = None

    for value in reversed(values):
        head = ListNode(value, head)

    return head


assert middle_node(build_linked_list([])) is None
assert middle_node(build_linked_list([1])).value == 1
assert middle_node(build_linked_list([1, 2, 3])).value == 2
assert middle_node(build_linked_list([1, 2, 3, 4])).value == 3
assert middle_node(build_linked_list([1, 2, 3, 4, 5, 6])).value == 4
~~~

**Common mistake:** Advancing both pointers at the same speed. That does not create the required half-distance relationship.

---

## 6. Testing your solutions

Examples help explain a problem, but they are not enough to establish correctness. For each function, test:
- the smallest valid input;
- empty input, if allowed;
- one-element input;
- duplicates;
- negative values, if allowed;
- an absent target;
- already ordered and reverse-ordered data where relevant;
- boundary positions;
- invalid inputs, if the contract defines their behavior.

### A small unittest example

Python's standard unittest framework provides test cases and assertions. Add more test methods for the remaining exercises.

~~~python
import unittest


class TestEasyProblems(unittest.TestCase):
    def test_count_occurrences(self):
        self.assertEqual(count_occurrences([2, 1, 2], 2), 2)
        self.assertEqual(count_occurrences([], 2), 0)

    def test_two_sum(self):
        self.assertEqual(two_sum_indices([3, 3], 6), [0, 1])
        self.assertIsNone(two_sum_indices([1, 2], 9))

    def test_binary_search(self):
        self.assertEqual(binary_search([1, 3, 5], 3), 1)
        self.assertEqual(binary_search([], 3), -1)

    def test_valid_parentheses(self):
        self.assertTrue(is_valid_parentheses("([])"))
        self.assertFalse(is_valid_parentheses("([)]"))


if __name__ == "__main__":
    unittest.main()
~~~

Keep each exercise independently testable. Avoid mixing input prompts, printing, and algorithm logic inside the function being tested.

---

## 7. Pattern and complexity revision table

| Problem | Main pattern | Typical time | Auxiliary space |
|---|---|---:|---:|
| 1. Count target occurrences | Array traversal | O(n) | O(1) |
| 2. Second-largest distinct | Single pass | O(n) | O(1) |
| 3. Reverse words | String/list operations | O(c) | O(c) |
| 4. Anagram check | Frequency counting | O(a+b) expected | O(u) |
| 5. Two Sum | Hash map | O(n) expected | O(n) |
| 6. Move zeroes | Two-index in-place scan | O(n) | O(1) |
| 7. First unique character | Frequency map + scan | O(n) expected | O(u) |
| 8. Binary search | Binary search | O(log n) | O(1) |
| 9. Valid parentheses | Stack | O(n) | O(n) |
| 10. Merge sorted lists | Two pointers | O(m+n) | O(m+n) output |
| 11. Best single transaction | Running minimum | O(n) | O(1) |
| 12. Linked-list middle | Fast and slow pointers | O(n) | O(1) |

Here n is the relevant input length, c is the number of characters, a and b are string lengths, u is the number of distinct values or characters, and m and n are the two sorted-list lengths in Problem 10. Expected hash-map costs assume average constant-time hash-table operations.

---

## 8. Completion checklist

- [ ] Explain each problem's contract without reading the solution.
- [ ] Implement all 12 solutions without copying.
- [ ] Explain why each algorithm is correct.
- [ ] Distinguish a count from a membership check.
- [ ] Handle duplicates when distinct values are required.
- [ ] Use a dictionary to find a complement without reusing an index.
- [ ] Preserve order when modifying an array in place.
- [ ] Explain why binary search requires sorted input.
- [ ] Use a stack to enforce nested ordering.
- [ ] Merge sorted inputs without sorting them again.
- [ ] Track a running minimum while preserving buy-before-sell order.
- [ ] Explain why fast and slow pointers locate the middle node.
- [ ] State time and auxiliary-space complexity with assumptions.
- [ ] Test empty, smallest, duplicate, and boundary cases.

If a solution is difficult to explain, revisit the relevant concept instead of memorizing the code.

---

## 9. Further study in this repository

- [DSA Interview Preparation Home](../README.md)
- [01 — Interview Roadmap](../01%20Interview%20Roadmap/README.md)
- [02 — Python Coding Interview](../02%20Python%20Coding%20Interview/README.md)
- [03 — Pattern Recognition](../03%20Pattern%20Recognition/README.md)
- [Complete DSA — Practice Problems](../../Complete%20DSA/18%20DSA%20Practice%20Problems/README.md)
- [Complete DSA — Interview Patterns](../../Complete%20DSA/17%20Interview%20Patterns/README.md)
- [DSA for DevOps — Python Basics](../../DSA%20for%20DevOps/01%20Python%20Basics%20for%20DSA/README.md)

---

## 10. Sources and quality notes

Official Python documentation consulted for language, container, and testing behavior:

- [Python Tutorial — Data Structures](https://docs.python.org/3/tutorial/datastructures.html) — list operations, dictionaries, and sets.
- [Python — Built-in Types](https://docs.python.org/3/library/stdtypes.html) — built-in sequence and mapping types and string operations.
- [Python — collections](https://docs.python.org/3/library/collections.html) — Counter and other specialized containers.
- [Python — unittest](https://docs.python.org/3/library/unittest.html) — test cases and assertion methods.

These references document Python behavior and APIs. The problem statements, solution choices, and algorithm explanations are repository learning material; the documentation links are not presented as proof of each algorithm's correctness.

**Verified facts:** The linked official Python documentation describes the language features and standard-library APIs used in the examples.

**Repository design decision:** The 12 exercises are classified as an easy progression and include worked solutions so learners can compare their reasoning after attempting each problem.

**Demonstrations:** Code is educational. Reuse it only after checking that the real problem's constraints match the stated contracts and tests.

---

**Previous:** [03 — Pattern Recognition](../03%20Pattern%20Recognition/README.md)  
**Back to:** [DSA Interview Preparation](../README.md)
