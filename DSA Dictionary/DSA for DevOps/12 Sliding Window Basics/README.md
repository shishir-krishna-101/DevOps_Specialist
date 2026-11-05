# Sliding Window Basics in Python

A **sliding window** examines a consecutive group of items in a sequence. Instead of calculating every group from scratch, we move the window forward and reuse information from the previous window.

In this lesson, you will find the **largest sum of any fixed-size group of consecutive measurements** using a simple monitoring-style example.

> **Learning goal:** Understand how a fixed-size window moves through a list and how to update its total efficiently.

## 1. What is a sliding window?

Imagine a script collected these sample values:

~~~python
cpu_samples = [20, 30, 40, 10, 50, 60]
~~~

Suppose you want to inspect every group of **three consecutive samples**.

| Window | Values | Sum |
| --- | --- | ---: |
| 1 | 20, 30, 40 | 90 |
| 2 | 30, 40, 10 | 80 |
| 3 | 40, 10, 50 | 100 |
| 4 | 10, 50, 60 | 120 |

Each time the window moves one position to the right:
- One value leaves the window.
- One new value enters the window.

The largest sum is **120**, from the window **[10, 50, 60]**.

This is an algorithm exercise using sample data, not a monitoring tool or a production metric.

## 2. Why might a DevOps engineer learn this?

This pattern can help reason about consecutive observations, for example:
- Compare totals across fixed groups of monitoring samples.
- Find the busiest consecutive interval in a small sample list.
- Examine batches of recent event counts.
- Identify a high-activity segment in test data.

Real monitoring systems have their own sampling intervals, aggregation rules, missing-data behavior, and alert thresholds. Those details are outside this basic exercise.

## 3. First solution: calculate every window from scratch

A simple first approach is to calculate the sum of each slice separately.

~~~python
cpu_samples = [20, 30, 40, 10, 50, 60]
window_size = 3

for start in range(len(cpu_samples) - window_size + 1):
    window = cpu_samples[start:start + window_size]
    print(window, sum(window))
~~~

Output:

~~~text
[20, 30, 40] 90
[30, 40, 10] 80
[40, 10, 50] 100
[10, 50, 60] 120
~~~

### Understand the code

- len(cpu_samples) gives the number of samples.
- range(len(cpu_samples) - window_size + 1) produces each valid starting index.
- The slice cpu_samples[start:start + window_size] selects consecutive values.
- sum(window) calculates that window's total.

This approach is easy to understand, but adjacent windows repeat most of the same work.

### Complexity of the first solution

Let n be the number of samples and k be the window size.

There are n - k + 1 windows when 1 <= k <= n. Each slice contains k values, and summing it examines those values.

- **Time:** O((n - k + 1) × k), commonly simplified to O(n × k).
- **Extra space:** O(k) for the temporary slice.

## 4. Improve it: reuse the previous window sum

Compare the first two windows:

~~~text
Previous: [20, 30, 40]  sum = 90
Next:        [30, 40, 10] sum = 80
~~~

To move from the first window to the next:
1. Subtract the value that leaves: 20.
2. Add the value that enters: 10.

So:

~~~text
90 - 20 + 10 = 80
~~~

We do not need to add all three values again.

The rolling update is:

~~~text
new_window_sum = previous_window_sum - outgoing_value + incoming_value
~~~

## 5. Efficient fixed-size sliding-window solution

~~~python
def largest_window_sum(values, window_size):
    if window_size <= 0 or window_size > len(values):
        return None

    current_sum = sum(values[:window_size])
    largest_sum = current_sum
    largest_start = 0

    for right in range(window_size, len(values)):
        outgoing_index = right - window_size

        current_sum -= values[outgoing_index]
        current_sum += values[right]

        if current_sum > largest_sum:
            largest_sum = current_sum
            largest_start = outgoing_index + 1

    return largest_sum, largest_start


cpu_samples = [20, 30, 40, 10, 50, 60]
result = largest_window_sum(cpu_samples, 3)

print(result)
~~~

Output:

~~~text
(120, 3)
~~~

The result means:
- Largest sum: 120
- Starting index: 3
- Matching window: [10, 50, 60]

### Line-by-line explanation

1. Reject a window size that is zero, negative, or larger than the input list.
2. Calculate the sum of the first window once.
3. Store that sum as the best sum found so far.
4. Store the first window's starting index, zero.
5. Start the loop at the first index that enters after the initial window.
6. Calculate the index of the value leaving the window.
7. Subtract the outgoing value from the current total.
8. Add the incoming value to the current total.
9. If this window has a larger sum, update the best sum.
10. Store the starting index of the new best window.
11. Return the best sum and its starting index.

## 6. Dry run

Input:

~~~python
values = [20, 30, 40, 10, 50, 60]
window_size = 3
~~~

Initial window:

~~~text
[20, 30, 40]
20 + 30 + 40 = 90
largest_sum = 90
largest_start = 0
~~~

Move the window one position at a time:

| Move | Outgoing | Incoming | Current sum | Best sum | Best start |
| --- | ---: | ---: | ---: | ---: | ---: |
| Initial | — | — | 90 | 90 | 0 |
| 1 | 20 | 10 | 80 | 90 | 0 |
| 2 | 30 | 50 | 100 | 100 | 2 |
| 3 | 40 | 60 | 120 | 120 | 3 |

Final window: **[10, 50, 60]**

Final answer: **(120, 3)**.

## 7. Trace the window visually

~~~text
Values:  [20, 30, 40, 10, 50, 60]
          └───────┘
          Window 1

Values:  [20, 30, 40, 10, 50, 60]
              └───────┘
              Window 2

Values:  [20, 30, 40, 10, 50, 60]
                  └───────┘
                  Window 3

Values:  [20, 30, 40, 10, 50, 60]
                      └───────┘
                      Window 4
~~~

The window always contains three consecutive elements.

## 8. Edge cases

### Window size is zero or negative

A zero-sized or negative-sized window is not meaningful for this exercise. The function returns None.

### Window size is larger than the list

There are not enough values to create a full window. The function returns None.

### Window size is one

Each window contains one value, so the largest window sum is the largest list element.

~~~python
assert largest_window_sum([4, 9, 2], 1) == (9, 1)
~~~

### All values are negative

The function initializes the best sum from the first complete window rather than from zero.

~~~python
assert largest_window_sum([-8, -3, -6, -2], 2) == (-5, 2)
~~~

### Equal maximum sums

The function updates only when it finds a strictly larger sum. If multiple windows have the same maximum sum, it returns the **first** one.

~~~python
assert largest_window_sum([5, 0, 5, 0], 2) == (5, 0)
~~~

### Empty input

An empty list cannot contain a valid positive-size window, so the function returns None.

~~~python
assert largest_window_sum([], 1) is None
~~~

## 9. Test the complete function

Copy the function from Section 5, then run these checks:

~~~python
assert largest_window_sum([20, 30, 40, 10, 50, 60], 3) == (120, 3)
assert largest_window_sum([4, 9, 2], 1) == (9, 1)
assert largest_window_sum([-8, -3, -6, -2], 2) == (-5, 2)
assert largest_window_sum([5, 0, 5, 0], 2) == (5, 0)
assert largest_window_sum([], 1) is None
assert largest_window_sum([1, 2], 3) is None
assert largest_window_sum([1, 2], 0) is None

print("Sliding-window tests passed.")
~~~

Expected output:

~~~text
Sliding-window tests passed.
~~~

## 10. Time and space complexity

Let n be the number of input values and k be the window size.

| Approach | Time complexity | Extra space |
| --- | --- | --- |
| Recalculate each slice | O((n - k + 1) × k), or O(n × k) | O(k) for a temporary slice |
| Rolling sum | O(n) | O(1) |

The efficient version calculates the first window once. Each following window uses one subtraction and one addition, so each step does constant work.

The rolling-sum function stores only a fixed number of variables; it does not create a new slice for each window.

## 11. Common beginner mistakes

1. **Recalculating every window:** This repeats work that can be reused from the previous window.
2. **Using the wrong outgoing index:** For a window ending at right, the outgoing index is right - window_size.
3. **Forgetting to add the incoming value:** The rolling total must subtract the old value and add the new one.
4. **Using range(len(values) - window_size):** This misses the final valid window. Include + 1 when enumerating starting positions.
5. **Starting the best sum at zero:** This gives an incorrect answer when all values are negative.
6. **Ignoring invalid window sizes:** Check that the window size is positive and does not exceed the input length.
7. **Mixing up index and value:** The function returns the starting index as well as the sum.

## 12. Beginner exercises

Try each exercise before opening its solution. Use the function from Section 5 as your starting point.

### Exercise 1: Find the largest sum

Find the largest sum of any two consecutive values in:

~~~python
values = [3, 8, 2, 10, 4]
~~~

<details>
<summary>Solution</summary>

~~~python
assert largest_window_sum([3, 8, 2, 10, 4], 2) == (12, 2)
~~~

The window [2, 10] has the largest sum.

</details>

### Exercise 2: Use a window size of one

Find the largest one-item window in:

~~~python
values = [7, 12, 5, 9]
~~~

<details>
<summary>Solution</summary>

~~~python
assert largest_window_sum([7, 12, 5, 9], 1) == (12, 1)
~~~

</details>

### Exercise 3: Work with negative values

Find the largest sum of two consecutive values in:

~~~python
values = [-5, -2, -8, -1]
~~~

<details>
<summary>Solution</summary>

~~~python
assert largest_window_sum([-5, -2, -8, -1], 2) == (-3, 0)
~~~

</details>

### Exercise 4: Find the first window

Print the first window of size three from:

~~~python
values = [11, 22, 33, 44, 55]
window_size = 3

first_window = values[:window_size]
print(first_window)
~~~

Output:

~~~text
[11, 22, 33]
~~~

</details>

### Exercise 5: Explain the rolling update

A previous window sum is 75. The outgoing value is 15, and the incoming value is 25. Calculate the next window sum.

<details>
<summary>Solution</summary>

~~~python
next_sum = 75 - 15 + 25
print(next_sum)
~~~

Output:

~~~text
85
~~~

</details>

## 13. Interview questions

1. What is a sliding window?
2. What makes a window a fixed-size window?
3. Why does recalculating every window sum repeat work?
4. Which value leaves the window when it moves one position to the right?
5. Which value enters the window?
6. Why do we initialize the best sum from the first complete window?
7. What is the time complexity of the rolling-sum solution?
8. What should a function do when the window size is invalid?

## 14. Completion checklist

- [ ] I can explain a sliding window using a list of numbers.
- [ ] I can identify the values in each consecutive window.
- [ ] I can calculate a window sum.
- [ ] I understand why recalculating each window repeats work.
- [ ] I can subtract the outgoing value and add the incoming value.
- [ ] I can implement a rolling sum in Python.
- [ ] I can handle invalid window sizes.
- [ ] I can explain the difference between O(n × k) and O(n) for this exercise.

## 15. Official Python references

These references document the Python features used in this lesson. The sliding-window algorithm and its complexity analysis are teaching material in this repository, not claims made by the Python documentation.

- [Python Standard Library — Common Sequence Operations](https://docs.python.org/3/library/stdtypes.html#common-sequence-operations): indexing, slicing, and len().
- [Python Standard Library — sum()](https://docs.python.org/3/library/functions.html#sum): summing values from an iterable.
- [Python Tutorial — The range() Function](https://docs.python.org/3/tutorial/controlflow.html#the-range-function): generating integer sequences for loops.
- [Python Tutorial — for Statements](https://docs.python.org/3/tutorial/controlflow.html#for-statements): iterating over values.

## Navigation

- Previous: [11 Queues](../11%20Queues/README.md)
- Also study: [09 Searching](../09%20Searching/README.md)
- [DSA for DevOps with Python — Roadmap](../README.md)

---

**Next step:** Try the exercises without looking at the solutions, then explain why the rolling sum needs only one subtraction and one addition per move.
