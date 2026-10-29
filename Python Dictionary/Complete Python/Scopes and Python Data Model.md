# Scopes and Python Data Model

Python becomes much easier to reason about once you separate **names**, **objects**, **namespaces**, and **scopes**. These concepts explain why assignments do not copy objects, why mutable values can appear to change through another name, why `global` and `nonlocal` exist, and how Python decides which name a piece of code refers to.

This topic is especially useful for automation code, where configuration objects, command results, collections, and reusable state are frequently passed between functions.

> **Source note:** The technical claims in this guide are based primarily on the official Python documentation. Examples marked as demonstrations are repository teaching examples, not claims about an external system.

## 1. The core mental model

Python programs work with **objects** and **names**.

- An **object** represents data or other program entities.
- A **name** is bound to an object.
- A **namespace** maps names to objects.
- A **scope** is the textual region where a namespace is directly accessible for unqualified name lookup.

A useful mental model is:

```text
name  ─────► object
             │
             ├── type
             ├── identity
             └── value
```

Do not describe Python variables as simple boxes that permanently contain values. For ordinary Python code, it is more accurate to think in terms of **names bound to objects**.

## 2. Objects have identity, type, and value

The Python data model states that every object has:

- **identity** — the identity of that particular object;
- **type** — which determines the operations the object supports and possible values;
- **value** — the data represented by the object.

An object's identity does not change during its lifetime. The built-in `id()` function returns an integer representing an object's identity, and `is` compares object identity.

```python
a = []
b = a

print(a is b)   # True
print(id(a) == id(b))  # True
```

The exact numeric value returned by `id()` should not be treated as a portable memory address. The Python documentation explicitly identifies a memory-address interpretation as a **CPython implementation detail**.

### `is` versus `==`

Use:

- `is` when identity itself matters;
- `==` when comparing values for equality.

For example:

```python
items = []
other_items = []

print(items == other_items)  # True: equal values
print(items is other_items)  # False: different objects
```

For ordinary application logic, do not replace value comparisons with `is`.

A common identity check is:

```python
result = None

if result is None:
    print("No result")
```

## 3. Assignment binds names; it does not copy objects

Consider:

```python
config = {"region": "ap-south-1"}
alias = config
```

Both names refer to the same dictionary object.

```text
config ─┐
        ├──► {"region": "ap-south-1"}
alias  ─┘
```

Changing the mutable object through one name is visible through the other:

```python
config = {"region": "ap-south-1"}
alias = config

alias["region"] = "us-east-1"

print(config)
# {'region': 'us-east-1'}
```

This is **aliasing**.

The assignment did not make a second dictionary. It created another binding to the existing object.

If you actually need a separate object, choose an appropriate copying strategy for the data structure and its required semantics. Do not assume assignment performs a copy.

## 4. Mutable and immutable objects

The data model distinguishes between **mutable** and **immutable** objects.

Examples of immutable types include numbers, strings, and tuples. Lists and dictionaries are mutable.

The important distinction is whether the object's value can change after creation.

```python
name = "prod"
name = "staging"
```

The second assignment does not mutate the original string. The name is rebound to another string object.

By contrast:

```python
servers = ["api-1"]
servers.append("api-2")
```

The list object itself is mutated.

### Immutable containers can contain mutable objects

A tuple is immutable as a container, but it can contain a reference to a mutable object:

```python
servers = ["api-1"]
record = ("production", servers)

servers.append("api-2")

print(record)
# ('production', ['api-1', 'api-2'])
```

The tuple's collection of contained references has not changed, but the referenced list has changed.

## 5. Namespaces

A **namespace** is a mapping from names to objects.

Python has several important namespaces, including:

- the built-in namespace;
- a module's global namespace;
- a function invocation's local namespace;
- a class namespace;
- object attributes, which form a namespace-like set of names accessed through an object.

Namespaces have different lifetimes and purposes.

For example, when you write:

```python
region = "production"
```

at module level, `region` becomes a name in that module's namespace.

Inside a function:

```python
def deploy():
    target = "production"
```

`target` is local to that function invocation unless other scope rules apply.

## 6. Scope and name lookup

A **scope** is a textual region where a namespace is directly accessible.

For ordinary function code, name lookup proceeds through nested scopes. The common teaching shorthand is **LEGB**:

```text
L — Local
E — Enclosing function scopes
G — Global/module scope
B — Built-in scope
```

Example:

```python
region = "global"


def outer():
    region = "enclosing"

    def inner():
        region = "local"
        print(region)

    inner()

outer()
```

`inner()` finds its local `region` first.

The official execution model describes this more precisely in terms of bindings, code blocks, scopes, and the nearest enclosing scope.

## 7. Assignment affects which scope receives a binding

Without a `global` or `nonlocal` declaration, a name bound inside a function is local to that function's scope.

```python
region = "production"


def change_region():
    region = "staging"
    print(region)

change_region()
print(region)
```

The module-level `region` remains unchanged.

The assignment inside the function creates a local binding rather than rebinding the module-level name.

## 8. The `global` statement

`global` tells Python that references and assignments to specified names in the current block should use the module-level global binding.

```python
region = "production"


def change_region():
    global region
    region = "staging"

change_region()
print(region)
# staging
```

Use global state sparingly. In automation programs, passing configuration or state explicitly is often easier to test and reason about than modifying module-level state.

## 9. The `nonlocal` statement

`nonlocal` is used inside a nested function when you want to rebind a name in an enclosing **function** scope.

```python
def make_counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment

counter = make_counter()
print(counter())  # 1
print(counter())  # 2
```

The nested function retains access to the enclosing scope's binding.

`nonlocal` does not refer to the module-level global scope. It refers to an existing binding in an enclosing function scope.

## 10. A subtle scope rule: binding anywhere in a function

Python determines local bindings for a function block based on name-binding operations in that block.

This can produce a surprising error:

```python
region = "production"


def show_region():
    print(region)
    region = "staging"
```

The assignment makes `region` a local name for the function block. The earlier reference therefore attempts to use the local binding before it has been given a value, resulting in `UnboundLocalError`.

The key lesson is:

> A name being visible outside a function does not mean an assignment to that same name inside the function will modify the outer binding.

When this behavior matters, make the intended scope explicit with function arguments, return values, `global`, or `nonlocal` as appropriate.

## 11. Functions and closures

A nested function can use a name from an enclosing function scope even after the enclosing function has returned.

```python
def make_prefixer(prefix):
    def add_prefix(value):
        return f"{prefix}{value}"

    return add_prefix

prefixer = make_prefixer("prod-")
print(prefixer("api"))
# prod-api
```

The nested function retains access to the relevant enclosing binding.

Closures become useful when building reusable behavior, factories, callbacks, and small stateful helpers. They should be understood before relying heavily on them in automation frameworks.

## 12. Class namespaces and scopes

A class definition creates and executes a class definition block. The resulting class namespace becomes part of the class's attributes.

```python
class Service:
    environment = "production"
```

Here, `environment` is defined in the class namespace and can be accessed through the class:

```python
print(Service.environment)
```

A critical distinction is that the class namespace is **not** simply an enclosing function scope for methods.

```python
class Service:
    environment = "production"

    def show(self):
        return environment
```

The method does not automatically resolve `environment` as a local name from the class namespace. Use an explicit class or instance reference when that is what you mean:

```python
class Service:
    environment = "production"

    def show(self):
        return self.environment
```

This distinction is important when designing classes for automation tooling.

## 13. Attribute access

An expression such as:

```python
service.region
```

uses attribute access: `region` is an attribute of the object referred to by `service`.

Attributes can be readable or writable depending on the object's behavior.

For example:

```python
class Service:
    pass

service = Service()
service.region = "ap-south-1"

print(service.region)
```

Python's data model defines a much richer attribute-access system, including special methods and descriptors. Those advanced mechanisms are useful later, but basic DevOps automation generally requires first understanding names, objects, classes, and ordinary attribute access.

## 14. The Python data model

The Python data model describes how objects behave and how the language interacts with them.

This includes concepts such as:

- object identity, type, and value;
- standard built-in types;
- attribute access;
- callable objects;
- special method names such as `__repr__`, `__str__`, and `__eq__`;
- iteration and other protocols;
- context management;
- numeric and comparison behavior.

A useful engineering perspective is that Python syntax often delegates behavior to an object's type and its data-model protocols.

For example, an expression such as:

```python
value + other
```

is not merely a special case in the language. Python's object model provides mechanisms for types to define operations such as arithmetic through special methods.

This is why understanding the data model helps explain how built-in syntax works with user-defined classes.

## 15. Object lifecycle and external resources

Python objects are not explicitly destroyed by a `destroy()` operation in normal application code. When objects become unreachable, an implementation may reclaim them through garbage collection.

The official documentation warns against depending on immediate finalization. This matters for resources such as open files or other external resources.

For resource management, use explicit mechanisms such as:

```python
with open("config.txt") as file:
    contents = file.read()
```

The `with` statement provides deterministic resource-management structure without relying on an object's eventual garbage collection.

> **Important:** CPython's current memory-management behavior is an implementation detail. Do not write portable Python automation that depends on a particular garbage-collection timing behavior.

## 16. DevOps automation examples

These concepts appear frequently in practical automation.

### Configuration aliasing

```python
def add_default(config):
    config["timeout"] = 30

settings = {"region": "ap-south-1"}
add_default(settings)
```

The function receives access to the same dictionary object. If that mutation is not intended, design the function to avoid modifying the caller's object or explicitly create the required copy.

### Passing state instead of using globals

Prefer:

```python
def deploy(service, environment):
    print(f"Deploying {service} to {environment}")
```

over hidden global state when the value is part of the operation's input.

This is a **repository design recommendation**, not a Python language requirement.

### Explicit object boundaries

For automation code, make it clear whether a function:

- reads an object;
- mutates an object;
- returns a new object;
- maintains private/enclosed state.

Clear ownership and mutation rules make scripts easier to test and safer to reuse.

## 17. Common mistakes

### Mistake 1: Treating assignment as copying

```python
b = a
```

does not mean "make a copy of `a`".

### Mistake 2: Using `is` for ordinary value comparison

Prefer:

```python
if status == "ready":
    ...
```

over:

```python
if status is "ready":
    ...
```

when the intention is value equality.

### Mistake 3: Assuming class variables are function-scope variables

Methods do not automatically get the class namespace as an enclosing lexical scope. Access class or instance state explicitly.

### Mistake 4: Using globals for every piece of shared state

Global state can make tests and reasoning harder. Prefer explicit parameters and return values when they express the dependency clearly.

### Mistake 5: Depending on garbage-collection timing

Close files and release external resources explicitly. Do not rely on an object eventually becoming unreachable.

### Mistake 6: Describing all Python implementations as CPython

If a statement depends on CPython internals, label it as such. Python language behavior and CPython implementation details are not interchangeable.

## 18. Practical exercises

### Exercise 1 — Identity versus equality

Create two separate lists with the same contents. Verify:

- `==` reports equal values;
- `is` reports different identities.

Then assign one list to another name and verify the identity relationship.

### Exercise 2 — Mutation through an alias

Create a dictionary representing deployment configuration. Bind it to a second name and mutate the dictionary through the second name. Explain why the first name observes the change.

### Exercise 3 — Scope debugging

Write a function that attempts to read a module-level variable and then assigns to a local variable with the same name. Observe the resulting exception and explain why it occurs.

### Exercise 4 — `nonlocal`

Implement a small counter factory using a nested function and `nonlocal`. Explain which namespace stores the counter binding.

### Exercise 5 — Class namespace

Create a class with a class attribute and a method. Demonstrate the difference between accessing the attribute through the class and through an instance.

### Exercise 6 — Resource management

Write a small file-reading example using `with`. Explain why explicit resource management is preferable to relying on garbage-collection timing.

## 19. Knowledge checks

Before moving on, you should be able to answer:

1. What is the difference between a name and an object?
2. What are an object's identity, type, and value?
3. When should `is` be used instead of `==`?
4. Why does `b = a` not copy the object referred to by `a`?
5. What is aliasing?
6. Which common Python types are mutable and which are immutable?
7. What is a namespace?
8. What does scope mean in Python?
9. What does the LEGB shorthand describe?
10. Why can an assignment inside a function cause `UnboundLocalError` for an earlier reference to the same name?
11. What does `global` change?
12. What does `nonlocal` change?
13. Why is a class namespace different from an enclosing function scope?
14. Why should automation code not depend on garbage-collection timing?
15. Which statements in your own code are Python language guarantees, and which depend on CPython implementation details?

## 20. Completion checklist

You are ready to continue when you can:

- explain names, bindings, and objects without describing assignment as copying;
- distinguish identity from equality;
- explain mutable versus immutable objects;
- identify common namespaces and scopes;
- trace ordinary name lookup through local, enclosing, global, and built-in scopes;
- explain `global` and `nonlocal`;
- diagnose a basic `UnboundLocalError` caused by local binding rules;
- explain why class attributes are not ordinary enclosing function variables;
- recognize aliasing in automation code;
- use explicit resource-management constructs rather than depending on garbage-collection timing;
- distinguish Python language behavior from CPython implementation details.

## 21. Authoritative references

- Python Tutorial — Classes:  
  https://docs.python.org/3/tutorial/classes.html
- Python Language Reference — Data model:  
  https://docs.python.org/3/reference/datamodel.html
- Python Language Reference — Execution model:  
  https://docs.python.org/3/reference/executionmodel.html

### Source boundary

**Verified fact:** Definitions and language behavior in this guide are based on the official Python documentation above.

**Repository design decision:** The DevOps-oriented recommendations, learning order, exercises, and examples are structured for this repository and are not presented as official Python guidance.

**Demonstration:** Code snippets in this guide are teaching examples intended to illustrate the documented concepts.
