# 13 — Divide and Conquer

Divide and conquer is an algorithm design technique that solves a problem by breaking it into smaller subproblems, solving those subproblems, and combining their results.

It is useful when a problem can be decomposed into smaller instances that are easier to solve and whose answers can be combined into an answer for the original problem.

Classic examples include merge sort, quicksort, binary search, and many geometric and numerical algorithms.

---

## Learning objectives

By the end of this chapter, you should be able to:

- Explain the divide, conquer, and combine stages.
- Recognize when a problem can be divided into smaller subproblems.
- Trace recursive divide-and-conquer algorithms.
- Implement merge sort and binary search.
- Understand the role of partitioning in quicksort.
- Write and solve basic recurrence relations.
- Analyze time and auxiliary-space complexity.
- Recognize common implementation errors and unsuitable problem decompositions.

## 1. What is divide and conquer?

A divide-and-conquer algorithm usually has three stages:

1. **Divide:** break the original problem into smaller subproblems.
2. **Conquer:** solve each subproblem, often recursively.
3. **Combine:** use the subproblem results to construct the answer to the original problem.

A base case handles a problem small enough to solve directly, without further recursive calls.

A generic outline:

    def divide_and_conquer(problem):
        if is_small_enough(problem):
            return solve_directly(problem)

        smaller_problems = divide(problem)
        smaller_results = [
            divide_and_conquer(subproblem)
            for subproblem in smaller_problems
        ]
        return combine(smaller_results)

This is a conceptual template, not a universal implementation. The correct division and combination steps depend on the problem.

### A simple mental model

Imagine sorting a large collection of values:

- Divide the collection into smaller collections.
- Sort each smaller collection.
- Merge the sorted collections into one sorted result.

The merge step is what turns the smaller answers into the complete answer.

## 2. Divide and conquer vs other approaches

| Technique | Main idea | Typical use |
|---|---|---|
| Divide and conquer | Solve smaller subproblems and combine results | Merge sort |
| Greedy | Make a safe local choice at each step | Activity selection |
| Dynamic programming | Reuse solutions to overlapping subproblems | Knapsack variants |
| Backtracking | Explore choices and undo them when needed | Permutations |
| Brute force | Enumerate candidate solutions directly | Small input baselines |

The categories can overlap in their use of recursion. The distinction is the structure of the problem and how subproblem results are used.

A common divide-and-conquer pattern creates independent subproblems. Dynamic programming is especially useful when the same subproblems recur and their results can be stored and reused.

## 3. Merge sort

Merge sort is a classic divide-and-conquer sorting algorithm.

### How it works

1. If the input has zero or one element, it is already sorted.
2. Split the input into two halves.
3. Recursively sort each half.
4. Merge the two sorted halves into one sorted sequence.

### Example

Start with:

    [8, 3, 5, 4, 7, 6, 1, 2]

Divide into smaller pieces until each piece contains one element:

    [8, 3, 5, 4]        [7, 6, 1, 2]
    [8, 3] [5, 4]      [7, 6] [1, 2]
    [8] [3] [5] [4]    [7] [6] [1] [2]

Then merge sorted pieces:

    [3, 8] [4, 5]      [6, 7] [1, 2]
    [3, 4, 5, 8]      [1, 2, 6, 7]
    [1, 2, 3, 4, 5, 6, 7, 8]

### Python implementation

    def merge(left, right):
        """Merge two already-sorted lists into a new sorted list."""
        result = []
        left_index = 0
        right_index = 0

        while left_index < len(left) and right_index < len(right):
            if left[left_index] <= right[right_index]:
                result.append(left[left_index])
                left_index += 1
            else:
                result.append(right[right_index])
                right_index += 1

        result.extend(left[left_index:])
        result.extend(right[right_index:])
        return result


    def merge_sort(values):
        """Return a new list containing values in sorted order."""
        if len(values) <= 1:
            return values[:]

        middle = len(values) // 2
        left_sorted = merge_sort(values[:middle])
        right_sorted = merge_sort(values[middle:])

        return merge(left_sorted, right_sorted)


    numbers = [8, 3, 5, 4, 7, 6, 1, 2]
    print(merge_sort(numbers))

### Why the merge step is linear

Assume both input lists are already sorted.

- Compare the first unprocessed element of each list.
- Append the smaller one.
- Advance the index for the list from which the element was taken.
- Continue until one list is exhausted.
- Append the remaining elements.

Each element is appended once, so merging lists with a total of n elements takes O(n) time.

### Complexity

For merge sort:

- Recursion divides the input into halves.
- There are approximately log₂(n) levels of division.
- At each level, merging processes a total of n elements.

Therefore:

- Time: O(n log n) in best, average, and worst cases for the standard implementation.
- Auxiliary storage: O(n) for merged output at a level, plus recursion-stack overhead.
- Recursion depth: O(log n) for balanced halving.

The implementation above also creates list slices. Those slices allocate additional lists; account for this when analyzing its actual memory behavior. Other implementations can use index ranges to avoid repeated slicing.

### Stability

A sorting algorithm is stable when equal-key elements retain their relative input order.

In the implementation above, the merge chooses from the left list when values compare equal (using <=). This preserves the relative order of equal values across the two halves, assuming the recursive sorts are stable.

Stability matters when sorting records by multiple fields in successive passes.

## 4. Quicksort

Quicksort is another divide-and-conquer sorting algorithm. Instead of merging sorted halves, it partitions values around a pivot.

### How it works

1. Choose a pivot value.
2. Partition the other values into groups relative to the pivot.
3. Recursively sort the partitions.
4. Combine the ordered partitions around the pivot.

The exact partition procedure depends on the implementation.

### Conceptual example

Input:

    [7, 2, 9, 4, 3]

Suppose the pivot is 4. A partition may place smaller values on one side and larger values on the other:

    [2, 3]  4  [7, 9]

Then the left and right partitions are sorted recursively.

### A clear Python implementation

This version creates new lists for readability. It is not an in-place implementation.

    def quicksort(values):
        if len(values) <= 1:
            return values[:]

        pivot = values[len(values) // 2]

        smaller = [value for value in values if value < pivot]
        equal = [value for value in values if value == pivot]
        larger = [value for value in values if value > pivot]

        return quicksort(smaller) + equal + quicksort(larger)


    numbers = [7, 2, 9, 4, 3, 4]
    print(quicksort(numbers))

This version handles duplicate values by collecting values equal to the pivot in a separate list.

### Complexity

The partitioning work is linear per recursive level, but the total depends on how balanced the partitions are.

- Best case: O(n log n), when partitions are reasonably balanced.
- Average case: O(n log n) under common assumptions about input and pivot behavior.
- Worst case: O(n²), when partitions repeatedly become highly unbalanced.
- The list-building implementation above uses additional memory for partition lists and recursive results.

An in-place quicksort can reduce auxiliary allocations, but its implementation is more involved and still needs careful pivot and partition handling.

### Pivot selection

Pivot selection influences partition balance. Common strategies include:

- First element.
- Last element.
- Middle-position element.
- Randomized pivot.
- Median-of-three selection.

No pivot strategy should be described as guaranteeing balanced partitions for every possible input unless that guarantee has actually been established.

## 5. Binary search as divide and conquer

Binary search repeatedly halves a sorted search range.

It is covered in detail in the Searching chapter, but it is also a compact example of divide and conquer.

### Requirements

For ordinary binary search:

- The search range must be ordered according to the comparison being used.
- Each step must be able to determine which half may contain the target.
- The algorithm must maintain a clear range invariant.

### Iterative implementation

    def binary_search(values, target):
        low = 0
        high = len(values) - 1

        while low <= high:
            middle = (low + high) // 2
            current = values[middle]

            if current == target:
                return middle
            if current < target:
                low = middle + 1
            else:
                high = middle - 1

        return -1


    print(binary_search([2, 4, 6, 8, 10], 8))

Each iteration removes roughly half of the remaining candidate positions.

- Time: O(log n).
- Auxiliary space: O(1) for this iterative implementation.

A recursive binary search expresses the same idea but uses call-stack space proportional to its recursion depth.

## 6. Recurrence relations

A recurrence describes the cost of a recursive algorithm in terms of the cost of smaller instances.

A common form is:

    T(n) = aT(n/b) + f(n)

Where:

- T(n) is the running time for input size n.
- a is the number of recursive subproblems.
- n/b is the approximate size of each subproblem.
- f(n) is the work outside the recursive calls, including division and combination.

This is a common model, not a formula that fits every recursive algorithm.

### Example: merge sort

Merge sort creates two subproblems of approximately half the size and performs linear merging work:

    T(n) = 2T(n/2) + O(n)

The resulting time complexity is O(n log n).

### Example: binary search

Binary search continues with one subproblem of half the size and performs constant comparison/update work per step:

    T(n) = T(n/2) + O(1)

The resulting time complexity is O(log n).

### Example: an unbalanced recursive split

If an algorithm repeatedly reduces a problem by only one element and performs linear work at each level, its recurrence may look like:

    T(n) = T(n-1) + O(n)

Summing the work across levels gives O(n²).

The shape of the recursion matters. Recursion alone does not imply logarithmic complexity.

## 7. The Master Theorem: a useful analysis tool

The Master Theorem provides asymptotic bounds for certain divide-and-conquer recurrences of the form:

    T(n) = aT(n/b) + f(n)

It compares the non-recursive work f(n) with the quantity:

    n^(log_b(a))

At a high level, common cases are:

- If f(n) grows polynomially slower than n^(log_b(a)), the recursive work dominates.
- If f(n) has the same asymptotic order, a logarithmic factor is introduced.
- If f(n) grows polynomially faster and the required regularity condition holds, the non-recursive work dominates.

Do not apply the theorem mechanically. First verify that the recurrence fits its form and that the conditions of the relevant case are satisfied. Some recurrences require other methods, such as recursion trees or substitution.

## 8. Recursion tree intuition

A recursion tree expands a recurrence into levels.

For merge sort:

- Level 0 contains one problem of size n.
- Level 1 contains two problems of size n/2.
- Level 2 contains four problems of size n/4.
- Each level performs a total of O(n) merge-related work.
- The number of levels is O(log n).

Total work is therefore O(n log n).

This method is useful when you want to see where the total work is performed rather than memorizing a theorem.

## 9. When divide and conquer is a good fit

Look for these characteristics:

- The problem can be divided into smaller instances of the same or a closely related problem.
- The subproblems can be solved independently, or their dependencies are manageable.
- The results can be combined efficiently.
- The division reduces the problem enough to make recursive progress.
- A simple base case can be defined.

Examples include:

- Sorting with merge sort.
- Sorting with quicksort.
- Searching a sorted range with binary search.
- Finding extrema in recursively divided ranges.
- Multiplying large numbers using recursive decomposition.
- Certain geometric problems that split points or regions.

The existence of smaller subproblems is not enough. If combining their answers is more expensive than solving the original problem directly, the approach may not be useful.

## 10. When divide and conquer may be unsuitable

Consider another design when:

- The subproblems overlap heavily and repeat the same work.
- The combine step is too expensive.
- The input cannot be split into meaningful smaller instances.
- The recursion depth or call overhead is unsuitable for the environment.
- An iterative or specialized algorithm is simpler and more efficient.

For overlapping subproblems, dynamic programming may avoid repeated work by caching results.

For example, a naive recursive Fibonacci implementation repeatedly calculates the same values. Memoization or bottom-up dynamic programming avoids that repetition.

## 11. Common implementation mistakes

- Forgetting a base case.
- Defining a recursive call that does not make the input smaller.
- Splitting the input incorrectly and dropping or duplicating elements.
- Combining subproblem results in the wrong order.
- Assuming all divide-and-conquer algorithms have O(n log n) time.
- Ignoring the cost of copying slices or creating temporary arrays.
- Confusing recursion-stack space with total auxiliary storage.
- Assuming quicksort always has balanced partitions.
- Applying binary search to unsorted input.
- Using a recurrence without defining what each term represents.

## 12. Testing divide-and-conquer algorithms

Test both the final result and the behavior around recursive boundaries.

For sorting algorithms, include:

- Empty input.
- One element.
- Two elements in ascending order.
- Two elements in descending order.
- Already sorted input.
- Reverse-sorted input.
- Duplicate values.
- Negative and positive values.
- All values equal.
- A length that is not a power of two.

For binary search, include:

- Empty input.
- Target at the first position.
- Target at the last position.
- Target absent and smaller than all values.
- Target absent and larger than all values.
- Duplicate values, with a clear definition of which matching index is acceptable.

Use a trusted baseline in tests when possible. For sorting, Python's built-in sorted() is a useful reference for checking output order.

## 13. Practice progression

### Beginner

1. Trace merge sort by hand on an eight-element list.
2. Implement a merge function for two sorted lists.
3. Implement merge sort and test empty, singleton, duplicate, and reverse-sorted inputs.
4. Write iterative binary search and explain how the search interval shrinks.

### Intermediate

5. Implement the readable list-based quicksort shown above.
6. Compare merge sort and quicksort on sorted, reverse-sorted, and duplicate-heavy inputs.
7. Write recurrences for merge sort and binary search.
8. Draw a recursion tree for a simple recurrence.

### Challenge

9. Implement an index-based merge sort that avoids creating slices at every recursive call.
10. Implement an in-place quicksort and document its partition invariant.
11. Compare recursive and iterative implementations where both are reasonable.
12. Identify a recursive algorithm with overlapping subproblems and redesign it using memoization.

## 14. Mini project — Recursive Data Processing Toolkit

<badge color="info">Repository demonstration</badge>

Build a small Python toolkit that demonstrates divide-and-conquer techniques on operational-style data.

### Suggested features

- Read a collection of numeric measurements or event durations.
- Sort a copy using merge sort.
- Search for a target value using binary search after sorting.
- Report minimum, maximum, and median values.
- Keep the original input unchanged.
- Compare custom algorithm results with Python's built-in sorting and a simple linear-search baseline.

### Suggested test cases

- Empty dataset.
- Single measurement.
- Repeated measurements.
- Negative and positive values.
- Already sorted input.
- Reverse-sorted input.
- Target found and target absent.
- Very small and moderately large datasets.

### Engineering notes

- Clearly distinguish demonstration algorithms from production library choices.
- Do not assume a hand-written implementation is faster than Python's built-in operations.
- If benchmarking, record the Python version, input shape, measurement method, and environment.
- Avoid presenting one timing run as a general performance conclusion.

This is a repository-designed learning project, not a claim about any particular monitoring or logging platform.

## 15. Interview questions

1. What are the divide, conquer, and combine stages?
2. What is a base case, and why is it necessary?
3. How does merge sort work?
4. Why is merge sort O(n log n)?
5. What makes quicksort's worst case O(n²)?
6. How does pivot selection affect quicksort?
7. Why does binary search require an ordered search range?
8. What is a recurrence relation?
9. How would you derive the merge-sort recurrence?
10. What does the Master Theorem help analyze?
11. When can a recursion tree be useful?
12. How is divide and conquer different from dynamic programming?
13. Why should slicing and temporary allocations be included in space analysis?
14. What tests help expose errors in recursive partitioning and merging?

## 16. Completion checklist

- [ ] I can explain divide, conquer, combine, and base case.
- [ ] I can trace merge sort on a small input.
- [ ] I can implement and test a merge operation.
- [ ] I understand merge sort's time and space costs.
- [ ] I can explain quicksort partitioning and its worst case.
- [ ] I understand binary search as repeated halving.
- [ ] I can write a recurrence for a simple recursive algorithm.
- [ ] I can use a recursion tree to estimate total work.
- [ ] I know when the Master Theorem is applicable.
- [ ] I account for temporary lists, arrays, and recursion-stack space.
- [ ] I can distinguish divide and conquer from dynamic programming.

## 17. Sources and quality notes

### Python implementation references

- [Python built-in sorted()](https://docs.python.org/3/library/functions.html#sorted)
- [Python list sorting](https://docs.python.org/3/library/stdtypes.html#list.sort)
- [Python tutorial: data structures](https://docs.python.org/3/tutorial/datastructures.html)

These official references document Python's built-in sequence and sorting behavior. They are implementation references, not proofs of the algorithmic complexity claims in this chapter.

### Content boundaries

- **Algorithmic concepts:** The chapter presents standard divide-and-conquer algorithms and their conventional asymptotic analyses under the stated models.
- **Python implementation:** The code examples are educational implementations intended for tracing and practice.
- **Repository design decision:** The Recursive Data Processing Toolkit is a learning project created for this repository.
- **Not implied:** A custom implementation is not necessarily preferable to a standard-library implementation in production. Benchmark and validate against the actual workload before making that decision.

---

[← Previous: 12 — Greedy Algorithms](../12%20Greedy%20Algorithms/README.md) · [Complete DSA home](../README.md) · [Next: 14 — Dynamic Programming](../14%20Dynamic%20Programming/README.md)
