# 10 — Stacks

A DevOps script may need to process nested or recently added information in reverse order:

- check whether brackets in a simple configuration-like expression are balanced;
- undo the most recently added operation in a small tool;
- process nested items from the most recently opened one;
- keep track of the most recent step in a workflow.

A **stack** is a data structure that follows **Last In, First Out (LIFO)**:

> The last item added is the first item removed.

Python lists can be used as stacks with `append()` and `pop()`.

This lesson explains the stack idea with simple examples, then uses a stack to check matching brackets.

> **Scope note:** The examples are small algorithm exercises. Bracket checking here handles only the bracket characters `()`, `[]`, and `{}`; it is not a complete parser for Python, YAML, JSON, shell scripts, or configuration files.

> **Source policy:** Python list and deque behavior described here is based on the official documentation linked at the end. The stack algorithms are repository teaching examples.

---

## 1. What is a stack?

Think of a stack of plates:

1. You place a new plate on the top.
2. You remove the plate from the top.
3. You do not normally remove the plate from the bottom first.

A stack works in the same way.

The two basic operations are:

- **Push:** add an item to the top.
- **Pop:** remove and return the item at the top.

The last item pushed is the first item popped.

## 2. Create a stack with a Python list

Python lists provide the operations needed for a basic stack:

- `append(item)` adds an item to the end of the list.
- `pop()` removes and returns the last item.

Example:

```python
stack = []

stack.append("first")
stack.append("second")
stack.append("third")

print(stack)

last_item = stack.pop()

print(last_item)
print(stack)
```

Output:

```text
['first', 'second', 'third']
third
['first', 'second']
```

The last value added was `"third"`, so it was the first value removed.

## 3. Understand push and pop

Let's make the operations explicit with functions:

```python
def push(stack, item):
    stack.append(item)


def pop_item(stack):
    if not stack:
        return None

    return stack.pop()


items = []

push(items, "build")
push(items, "test")
push(items, "deploy")

print(pop_item(items))
print(pop_item(items))
print(items)
```

Output:

```text
deploy
test
['build']
```

The `if not stack` check handles an empty stack before calling `pop()`.

Python's list `pop()` raises `IndexError` if the list is empty. In this teaching function, we choose to return `None` instead. That is a repository design decision for this example, not a special behavior of list.pop().

## 4. Dry run: follow the stack

Operations:

```text
push("build")
push("test")
push("deploy")
pop()
pop()
```

| Operation | Stack after operation | Returned value |
|---|---|---|
| Start | `[]` | — |
| push build | `['build']` | — |
| push test | `['build', 'test']` | — |
| push deploy | `['build', 'test', 'deploy']` | — |
| pop | `['build', 'test']` | deploy |
| pop | `['build']` | test |

The item at the right-hand end of the list is the top of the stack.

## 5. Look at the top without removing it

If the stack is not empty, the last item can be read using index `-1`:

```python
stack = ["build", "test", "deploy"]

if stack:
    print(stack[-1])
```

Output:

```text
deploy
```

This is often called **peek**: inspect the top item without removing it.

Do not access `stack[-1]` when the stack may be empty; an empty list has no last element.

## 6. Mini project — check matching brackets

A common introductory stack problem is to check whether brackets are balanced.

Examples:

| Input | Balanced? | Reason |
|---|---|---|
| `()` | Yes | Opening and closing pair |
| `([])` | Yes | Correctly nested pairs |
| `{[()]}` | Yes | Correct nesting order |
| `([)]` | No | Closing brackets do not match the most recent opening bracket |
| `((` | No | An opening bracket remains |
| `())` | No | A closing bracket has no matching opening bracket |

### Why does a stack help?

When an opening bracket appears, remember it by pushing it onto the stack.

When a closing bracket appears:

1. There must be an opening bracket on the stack.
2. The most recent opening bracket must match the closing bracket.
3. Remove that opening bracket from the stack.

At the end, the stack must be empty.

### Python implementation

```python
def brackets_are_balanced(text):
    stack = []

    matching_opening = {
        ")": "(",
        "]": "[",
        "}": "{"
    }

    opening_brackets = {"(", "[", "{"}

    for character in text:
        if character in opening_brackets:
            stack.append(character)

        elif character in matching_opening:
            if not stack:
                return False

            most_recent_opening = stack.pop()

            if most_recent_opening != matching_opening[character]:
                return False

    return len(stack) == 0


print(brackets_are_balanced("([])"))
print(brackets_are_balanced("([)]"))
print(brackets_are_balanced("{[()]}"))
```

Output:

```text
True
False
True
```

### Line-by-line explanation

- `stack = []` creates an empty stack for opening brackets.
- `matching_opening` maps each closing bracket to the opening bracket it requires.
- `opening_brackets` contains the three opening bracket characters.
- `for character in text:` examines each character from left to right.
- If the character is an opening bracket, `append()` pushes it onto the stack.
- If it is a closing bracket and the stack is empty, there is no opening bracket to match it, so return `False`.
- Otherwise, `pop()` removes the most recent opening bracket.
- If that bracket is not the expected matching type, return `False`.
- After processing all characters, return whether the stack is empty.

Characters that are not one of these six bracket characters are ignored by this particular exercise.

### Dry run: `{[()]}`

| Character | Action | Stack after action |
|---|---|---|
| `{` | Push opening brace | `['{']` |
| `[` | Push opening square bracket | `['{', '[']` |
| `(` | Push opening parenthesis | `['{', '[', '(']` |
| `)` | Pop `(`; it matches | `['{', '[']` |
| `]` | Pop `[`; it matches | `['{']` |
| `}` | Pop `{`; it matches | `[]` |

The stack is empty at the end, so the input is balanced.

### Complexity

Let `n` be the number of characters in the input string.

- **Time:** O(n), because each character is visited once and each bracket is pushed or popped at most once.
- **Additional space:** O(n) in the worst case, when all characters are opening brackets.

## 7. A stack for recent workflow steps

A stack can model a simple history where the most recently recorded step is examined first.

```python
recent_steps = []

recent_steps.append("checkout")
recent_steps.append("build")
recent_steps.append("test")

last_step = recent_steps.pop()

print(last_step)
print(recent_steps)
```

Output:

```text
test
['checkout', 'build']
```

This illustrates LIFO behavior. It does not implement a CI/CD workflow engine, retry policy, rollback system, or deployment controller.

## 8. Stack versus queue

A stack and a queue both store multiple items, but their removal order differs.

| Data structure | Rule | First item removed |
|---|---|---|
| Stack | LIFO — Last In, First Out | Most recently added |
| Queue | FIFO — First In, First Out | Earliest added |

Use a stack when the most recent item should be processed first. Use a queue when items should be processed in arrival order.

We will study queues in the next lesson.

## 9. Common mistakes

| Mistake | Why it is a problem | Better habit |
|---|---|---|
| Using `pop(0)` for a stack | Removes the first list item, not the top | Use `pop()` |
| Popping an empty list | Raises `IndexError` | Check whether the stack is empty |
| Reading `stack[-1]` when empty | Raises `IndexError` | Check before peeking |
| Removing the wrong opening bracket | Nested pairs may be matched incorrectly | Always compare with the most recently pushed bracket |
| Checking only the number of opening and closing brackets | Equal counts do not guarantee correct nesting | Check bracket type and order |
| Forgetting to check the final stack | Unmatched opening brackets may remain | Ensure the stack is empty at the end |
| Assuming this example parses a real config format | It ignores quoting, comments, and grammar | Use a parser designed for the actual format |

## 10. Practice exercises

Try these before opening the suggested solutions.

### Exercise 1 — Push items

Create an empty stack and push three job names onto it.

### Exercise 2 — Pop an item

Pop the most recently added item and print it.

### Exercise 3 — Peek

Print the top item without removing it. Handle the empty-stack case.

### Exercise 4 — Safe pop

Write a function that returns the popped item, or `None` if the stack is empty.

### Exercise 5 — Reverse the order

Use a stack to reverse the order of a list of strings.

### Exercise 6 — Balanced parentheses

Write a function that returns whether a string containing only `()` is balanced.

### Exercise 7 — Balanced brackets

Extend the solution to support `()`, `[]`, and `{}`.

### Exercise 8 — Trace a stack

For these operations, write the stack after each step:

```text
push("a")
push("b")
pop()
push("c")
pop()
```

---

## Suggested solutions

<details>
<summary>Exercise 1 — Push items</summary>

```python
stack = []

stack.append("build")
stack.append("test")
stack.append("deploy")

assert stack == ["build", "test", "deploy"]
```

</details>

<details>
<summary>Exercise 2 — Pop an item</summary>

```python
stack = ["build", "test", "deploy"]

last_item = stack.pop()

assert last_item == "deploy"
assert stack == ["build", "test"]
```

</details>

<details>
<summary>Exercise 3 — Peek</summary>

```python
def peek(stack):
    if not stack:
        return None

    return stack[-1]


assert peek(["build", "test"]) == "test"
assert peek([]) is None
```

</details>

<details>
<summary>Exercise 4 — Safe pop</summary>

```python
def safe_pop(stack):
    if not stack:
        return None

    return stack.pop()


items = ["first", "last"]

assert safe_pop(items) == "last"
assert safe_pop(items) == "first"
assert safe_pop(items) is None
```

</details>

<details>
<summary>Exercise 5 — Reverse the order</summary>

```python
def reverse_with_stack(items):
    stack = []

    for item in items:
        stack.append(item)

    reversed_items = []

    while stack:
        reversed_items.append(stack.pop())

    return reversed_items


assert reverse_with_stack(["build", "test", "deploy"]) == [
    "deploy", "test", "build"
]
assert reverse_with_stack([]) == []
```

</details>

<details>
<summary>Exercise 6 — Balanced parentheses</summary>

```python
def parentheses_are_balanced(text):
    stack = []

    for character in text:
        if character == "(":
            stack.append(character)
        elif character == ")":
            if not stack:
                return False
            stack.pop()

    return not stack


assert parentheses_are_balanced("()") is True
assert parentheses_are_balanced("(())") is True
assert parentheses_are_balanced(")(") is False
assert parentheses_are_balanced("(()") is False
```

</details>

<details>
<summary>Exercise 7 — Balanced brackets</summary>

```python
def brackets_are_balanced(text):
    stack = []
    pairs = {")": "(", "]": "[", "}": "{"}
    openings = {"(", "[", "{"}

    for character in text:
        if character in openings:
            stack.append(character)
        elif character in pairs:
            if not stack:
                return False
            if stack.pop() != pairs[character]:
                return False

    return not stack


assert brackets_are_balanced("{[()]}") is True
assert brackets_are_balanced("([)]") is False
assert brackets_are_balanced("(((") is False
assert brackets_are_balanced("())") is False
```

</details>

<details>
<summary>Exercise 8 — Trace a stack</summary>

| Operation | Stack |
|---|---|
| Start | `[]` |
| push a | `['a']` |
| push b | `['a', 'b']` |
| pop | `['a']` |
| push c | `['a', 'c']` |
| pop | `['a']` |

The values returned by the two pop operations are `"b"` and `"c"`.

</details>

## 11. Test the bracket checker

```python
def brackets_are_balanced(text):
    stack = []
    pairs = {")": "(", "]": "[", "}": "{"}
    openings = {"(", "[", "{"}

    for character in text:
        if character in openings:
            stack.append(character)
        elif character in pairs:
            if not stack:
                return False
            if stack.pop() != pairs[character]:
                return False

    return not stack


assert brackets_are_balanced("") is True
assert brackets_are_balanced("()") is True
assert brackets_are_balanced("([])") is True
assert brackets_are_balanced("{[()]}") is True
assert brackets_are_balanced("([)]") is False
assert brackets_are_balanced("(()") is False
assert brackets_are_balanced("())") is False
assert brackets_are_balanced("}") is False

print("All checks passed")
```

Expected output:

```text
All checks passed
```

Also test:

- empty input;
- only opening brackets;
- only closing brackets;
- mismatched bracket types;
- correctly nested brackets;
- extra non-bracket characters, remembering that this exercise ignores them.

## 12. Lesson completion checklist

- [ ] Explain LIFO in your own words.
- [ ] Explain push and pop.
- [ ] Implement a stack using a Python list.
- [ ] Safely handle an empty stack.
- [ ] Peek at the top item.
- [ ] Trace stack operations manually.
- [ ] Explain why `pop()` removes the last list element.
- [ ] Use a stack to reverse a sequence.
- [ ] Use a stack to check balanced brackets.
- [ ] Explain the time and additional-space complexity.
- [ ] Distinguish a stack from a queue.
- [ ] Explain the limitations of the bracket-checking example.

---

## Official Python documentation

1. [The Python Tutorial — Using Lists as Stacks](https://docs.python.org/3/tutorial/datastructures.html#using-lists-as-stacks) — explains LIFO behavior using list `append()` and `pop()`.
2. [Built-in Types — list.append()](https://docs.python.org/3/library/stdtypes.html#list.append) — reference for adding an item to a list.
3. [Built-in Types — list.pop()](https://docs.python.org/3/library/stdtypes.html#list.pop) — reference for removing and returning a list item.
4. [collections.deque](https://docs.python.org/3/library/collections.html#collections.deque) — documents a double-ended queue that supports appends and pops at either end.

For this beginner stack exercise, a list is sufficient. A deque is useful when a program needs efficient operations at both ends, particularly for queue-style processing.

The official documentation verifies Python's collection behavior. The bracket checker and workflow-step examples are repository teaching exercises.

---

**Previous:** [08 — Sorting](../08%20Sorting/README.md)

**Also see:** [09 — Searching](../09%20Searching/README.md)

**Next:** [11 — Queues](../11%20Queues/README.md) — process jobs in the order they arrive.
