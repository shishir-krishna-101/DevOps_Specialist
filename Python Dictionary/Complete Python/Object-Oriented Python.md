# Object-Oriented Python

Object-oriented programming (OOP) in Python is a way to bundle related state and behavior into objects. Python's class system supports classes, instances, inheritance, method overriding, multiple inheritance, and special methods.

> **Prerequisite:** Complete [Advanced Functions](Advanced%20Functions.md) and the functions/classes basics in [Python for DevOps](../Python%20for%20DevOps/README.md).

## 1. What you will learn

By the end of this topic, you should be able to:

- explain classes, instances, attributes, and methods
- distinguish class variables from instance variables
- understand how `self` works
- design constructors with `__init__`
- use inheritance and method overriding deliberately
- use `super()` when extending inherited behavior
- understand class methods and static methods
- use properties when controlled attribute access is useful
- understand Python's convention for non-public attributes
- implement useful special methods such as `__repr__` and `__eq__`
- recognize when a class is useful and when a simpler function or data structure is better

---

## 2. The mental model

A class defines a type. An instance is an object created from that type.

Think of the relationship as:

```text
Class
  │
  ├── defines behavior
  ├── defines shared attributes
  └── describes the structure/operations of instances
          │
          ├── instance A → its own state
          ├── instance B → its own state
          └── instance C → its own state
```

Python's documentation describes classes as a way of bundling data and functionality together. A class creates a new type, and instances can hold state and use methods defined by the class.

**Verified fact:** Python classes are created at runtime and can be modified after creation. See the official Python tutorial.

---

## 3. Your first class

```python
class Server:
    pass

server = Server()
```

`Server` is the class. `server` is an instance of that class.

A class definition creates a class object. Calling the class creates an instance.

```python
print(type(server))
```

The exact representation printed by `type()` is implementation/runtime output and should not be treated as a fixed teaching output.

---

## 4. Instance attributes

Instance attributes store state belonging to a particular object.

```python
class Server:
    pass

server = Server()
server.hostname = "api-01"
server.environment = "production"
```

Another instance can have different state:

```python
server_a = Server()
server_b = Server()

server_a.hostname = "api-01"
server_b.hostname = "worker-01"
```

The two instances are separate objects with separate instance attributes.

Python does not require instance attributes to be declared in a separate declaration block. They can be created by assignment.

---

## 5. `__init__` and object initialization

Most useful classes initialize their state in `__init__`.

```python
class Server:
    def __init__(self, hostname, environment):
        self.hostname = hostname
        self.environment = environment

server = Server("api-01", "production")
```

The arguments supplied during instantiation are passed to `__init__`.

### What is `self`?

`self` is the conventional name for the instance passed to an instance method.

```python
class Server:
    def __init__(self, hostname):
        self.hostname = hostname

    def describe(self):
        return self.hostname
```

When you call:

```python
server.describe()
```

the instance is supplied as the first argument to the underlying method function.

Conceptually:

```python
Server.describe(server)
```

This is a useful mental model for understanding methods. `self` is not a special keyword; it is the conventional parameter name used for the instance.

---

## 6. Methods

A method is a function defined in a class that can operate on an instance.

```python
class Server:
    def __init__(self, hostname, environment):
        self.hostname = hostname
        self.environment = environment

    def is_production(self):
        return self.environment == "production"
```

Usage:

```python
server = Server("api-01", "production")
print(server.is_production())
```

Methods are useful when behavior naturally belongs with the state represented by the object.

---

## 7. Instance variables vs class variables

This distinction is essential.

### Instance variable

Unique to each instance:

```python
class Server:
    def __init__(self, hostname):
        self.hostname = hostname
```

Each `Server` can have a different `hostname`.

### Class variable

Defined on the class and shared through the class unless an instance provides its own attribute with the same name.

```python
class Server:
    category = "compute"

    def __init__(self, hostname):
        self.hostname = hostname
```

Both instances can access `category`:

```python
first = Server("api-01")
second = Server("worker-01")

print(first.category)
print(second.category)
```

### Common mistake: mutable class state

Avoid using a mutable class variable when every instance should have its own collection.

Bad:

```python
class Server:
    tags = []
```

All instances can end up sharing the same list.

Prefer:

```python
class Server:
    def __init__(self):
        self.tags = []
```

**Verified fact:** The official Python tutorial demonstrates this exact distinction with class and instance variables and warns about accidentally sharing mutable class data.

---

## 8. Encapsulation in Python

Python does not enforce Java-style private instance variables through a `private` keyword.

Instead, Python uses naming conventions and name mangling.

### Single leading underscore

```python
class Server:
    def __init__(self, hostname):
        self._hostname = hostname
```

A leading underscore conventionally communicates:

> This is an implementation detail; external code should generally avoid depending on it.

This is a convention, not an access-control mechanism.

### Double leading underscore

```python
class Server:
    def __init__(self, token):
        self.__token = token
```

Names beginning with two underscores in a class can be subject to **name mangling**. This is primarily intended to help avoid accidental name collisions in subclasses, not to provide strong security or secrecy.

Never treat name mangling as a mechanism for protecting secrets.

---

## 9. Properties

A property lets an attribute-style interface execute method logic.

```python
class Server:
    def __init__(self, hostname):
        self._hostname = hostname

    @property
    def hostname(self):
        return self._hostname
```

Now callers can write:

```python
server.hostname
```

instead of explicitly calling a getter method.

A property can also define controlled assignment:

```python
class Server:
    def __init__(self, hostname):
        self.hostname = hostname

    @property
    def hostname(self):
        return self._hostname

    @hostname.setter
    def hostname(self, value):
        if not value:
            raise ValueError("hostname cannot be empty")
        self._hostname = value
```

This allows validation while keeping a natural attribute interface.

### When to use properties

Properties are useful when:

- a value needs validation
- a value is computed from other state
- you need to preserve an attribute-style API while adding logic

Do not create a property merely to wrap a plain attribute without a reason.

---

## 10. Inheritance

Inheritance allows a class to derive from another class.

```python
class Resource:
    def describe(self):
        return "generic resource"


class Server(Resource):
    pass
```

`Server` inherits behavior from `Resource`.

```python
server = Server()
print(server.describe())
```

Python supports multiple inheritance, meaning a class can have more than one base class.

Use inheritance when the relationship genuinely represents a reusable subtype relationship. Do not use inheritance simply to reuse a few unrelated functions.

---

## 11. Method overriding

A derived class can replace an inherited method with its own implementation.

```python
class Resource:
    def describe(self):
        return "generic resource"


class Server(Resource):
    def describe(self):
        return "server resource"
```

Now `Server().describe()` uses the overridden implementation.

This is **method overriding**.

---

## 12. `super()`

`super()` provides a way to access methods from a parent/next class in the method resolution order.

A common use is extending initialization:

```python
class Resource:
    def __init__(self, name):
        self.name = name


class Server(Resource):
    def __init__(self, name, hostname):
        super().__init__(name)
        self.hostname = hostname
```

The subclass reuses the base-class initialization and adds its own state.

Do not think of `super()` simply as “call my immediate parent.” Its behavior is defined in terms of Python's method resolution order, which becomes particularly important with multiple inheritance.

---

## 13. Multiple inheritance and MRO

Python permits multiple base classes:

```python
class Auditable:
    pass


class Deployable:
    pass


class Service(Auditable, Deployable):
    pass
```

When multiple inheritance is involved, Python uses a **method resolution order (MRO)** to determine where attribute and method lookup proceeds.

You can inspect it with:

```python
print(Service.__mro__)
```

The exact tuple contents depend on the class hierarchy you define.

### Practical guidance

Multiple inheritance is a powerful feature, but it can make designs harder to understand. For application and automation code, prefer simple inheritance or composition unless multiple inheritance provides a clear benefit.

---

## 14. Composition

Composition means an object contains another object instead of inheriting from it.

```python
class Logger:
    def log(self, message):
        print(message)


class Deployment:
    def __init__(self, logger):
        self.logger = logger

    def run(self):
        self.logger.log("deployment started")
```

`Deployment` does not need to be a subclass of `Logger`. It uses a `Logger` object.

Composition is often a strong choice when one component **uses** another component rather than **being a** specialized version of it.

### Mental model

```text
Inheritance:
Server IS-A Resource

Composition:
Deployment HAS-A Logger
```

These are design guidelines rather than Python language requirements.

---

## 15. Class methods

A class method receives the class as its implicit first argument.

```python
class Server:
    default_environment = "development"

    @classmethod
    def default(cls):
        return cls("localhost", cls.default_environment)

    def __init__(self, hostname, environment):
        self.hostname = hostname
        self.environment = environment
```

The conventional first parameter is `cls`.

A class method can be called through the class and is useful when the operation needs class-level information rather than a particular instance.

A common use is an alternate constructor:

```python
server = Server.default()
```

---

## 16. Static methods

A static method is a function stored in a class namespace that does not receive an implicit instance or class argument.

```python
class Server:
    @staticmethod
    def normalize_hostname(hostname):
        return hostname.strip().lower()
```

Usage:

```python
normalized = Server.normalize_hostname(" API-01 ")
```

A static method is useful when the operation is conceptually related to the class but does not need `self` or `cls`.

If a function has no meaningful relationship to the class, keeping it as a normal module-level function may be clearer.

---

## 17. Useful special methods

Python classes can define special methods that integrate objects with Python syntax and built-in operations.

### `__repr__`

Provide a useful representation for debugging and inspection.

```python
class Server:
    def __init__(self, hostname):
        self.hostname = hostname

    def __repr__(self):
        return f"Server(hostname={self.hostname!r})"
```

### `__str__`

Define a user-facing string representation when appropriate.

```python
class Server:
    def __init__(self, hostname):
        self.hostname = hostname

    def __str__(self):
        return self.hostname
```

### `__eq__`

Define equality semantics when the class needs value-based comparison.

```python
class Server:
    def __init__(self, hostname):
        self.hostname = hostname

    def __eq__(self, other):
        if not isinstance(other, Server):
            return NotImplemented
        return self.hostname == other.hostname
```

Special methods are part of Python's data model. Learn them as integration points with the language rather than as arbitrary magic names.

---

## 18. `isinstance()` and `issubclass()`

Use `isinstance()` to test whether an object is an instance of a class or compatible class hierarchy.

```python
server = Server("api-01")

if isinstance(server, Server):
    print("server object")
```

Use `issubclass()` to test relationships between classes.

```python
class Resource:
    pass


class Server(Resource):
    pass

print(issubclass(Server, Resource))
```

These checks can be useful, but avoid designing code that constantly branches on concrete types when polymorphism or a simpler interface would be clearer.

---

## 19. OOP for DevOps automation

Classes can be useful when automation code has persistent state and several related operations.

For example, a small deployment object might represent:

```text
Deployment
├── service
├── environment
├── configuration
└── operations
    ├── validate()
    ├── plan()
    └── execute()
```

A repository design might look like:

```python
class Deployment:
    def __init__(self, service, environment):
        self.service = service
        self.environment = environment

    def validate(self):
        ...

    def plan(self):
        ...

    def execute(self):
        ...
```

The exact implementation depends on the automation system. This example is a **repository design demonstration**, not a claim about any particular deployment platform.

### Keep side effects explicit

Methods that modify infrastructure, files, processes, or external systems should have names and interfaces that make their effects understandable.

Avoid vague methods such as:

```python
def process(self):
    ...
```

when the real operation is something specific such as:

```python
def write_manifest(self):
    ...
```

Clear naming is especially valuable in operational automation where code may be used during incidents.

---

## 20. Dataclasses are a separate tool

Python's `dataclasses` module can reduce boilerplate for classes that primarily represent data.

Example:

```python
from dataclasses import dataclass


@dataclass
class Server:
    hostname: str
    environment: str
```

Dataclasses are intentionally covered in the later **Dataclasses and Type Hints** topic. Do not assume every class should become a dataclass.

Use a normal class when the class's behavior, lifecycle, inheritance, or custom object semantics are central to the design.

---

## 21. Common mistakes

### Mistake 1: Forgetting `self`

Incorrect:

```python
class Server:
    def describe():
        return "server"
```

An instance method normally needs the instance parameter.

Correct:

```python
class Server:
    def describe(self):
        return "server"
```

### Mistake 2: Using a mutable class variable accidentally

```python
class Server:
    tags = []
```

If each server needs its own tags, initialize the list per instance.

### Mistake 3: Using inheritance only for code reuse

If the relationship is not conceptually a subtype relationship, composition may be clearer.

### Mistake 4: Making every field a property

Properties should solve a real interface or validation problem.

### Mistake 5: Treating `_name` as private security

A leading underscore is a convention. It does not provide security.

### Mistake 6: Building deep inheritance trees

Deep hierarchies can make behavior difficult to trace. Prefer simple, understandable relationships.

### Mistake 7: Putting infrastructure side effects everywhere

A class with every method directly changing external systems can become difficult to test. Separate pure transformation/validation logic from side effects where practical.

---

## 22. Practical exercise: service model

Build a `Service` class with:

- `name`
- `environment`
- `version`
- `enabled`

Add methods:

```text
is_production()
summary()
enable()
disable()
```

Then create two independent instances and verify that changing one does not change the other.

### Extension

Add a class variable describing the resource category.

---

## 23. Practical exercise: deployment inheritance

Create:

```text
Deployment
└── KubernetesDeployment
```

The base class should define a general interface such as:

```text
validate()
plan()
```

The derived class should override the methods with Kubernetes-specific demonstration behavior.

Do not connect this exercise to a real Kubernetes cluster yet. The goal is to practice class design and overriding without introducing external system complexity.

---

## 24. Practical exercise: composition

Create:

```text
Deployment
├── Logger
└── Config
```

The `Deployment` object should use the other objects rather than inherit from them.

Then compare the design with an inheritance-based version.

Ask:

- Which relationship is easier to explain?
- Which version is easier to replace or test?
- Which version has fewer assumptions about the other component?

---

## 25. Knowledge check

You should be able to answer these without looking at the examples:

1. What is the difference between a class and an instance?
2. What does `self` represent?
3. Why is `__init__` commonly used?
4. What is the difference between an instance variable and a class variable?
5. Why can a mutable class variable be dangerous?
6. What does inheritance provide?
7. What is method overriding?
8. What problem does `super()` help solve?
9. What is method resolution order?
10. When is composition preferable to inheritance?
11. What is the difference between `@classmethod` and `@staticmethod`?
12. What problem can a property solve?
13. What does a leading underscore communicate?
14. Why should name mangling not be treated as secret storage?
15. What role do special methods play in Python?

---

## 26. Completion checklist

You are ready to continue when you can:

- [ ] create classes and instances confidently
- [ ] initialize instance state with `__init__`
- [ ] explain how instance methods receive `self`
- [ ] distinguish class and instance attributes
- [ ] avoid accidental shared mutable class state
- [ ] use properties for appropriate interfaces
- [ ] explain inheritance and overriding
- [ ] use `super()` correctly in simple hierarchies
- [ ] explain the purpose of MRO
- [ ] choose composition when it better represents the relationship
- [ ] distinguish class methods from static methods
- [ ] implement useful `__repr__` or other special methods
- [ ] design a small stateful automation component without over-engineering it

---

## 27. Authoritative references

Primary sources used for this topic:

- Python Tutorial — Classes: https://docs.python.org/3/tutorial/classes.html
- Python Language Reference — Data model: https://docs.python.org/3/reference/datamodel.html
- Python Language Reference — Compound statements: https://docs.python.org/3/reference/compound_stmts.html

These sources establish the Python language behavior. The DevOps-oriented examples, design recommendations, exercises, and learning sequence in this file are repository teaching material.
