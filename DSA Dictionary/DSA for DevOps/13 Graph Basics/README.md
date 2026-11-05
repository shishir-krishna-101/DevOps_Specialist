# Graph Basics in Python

A **graph** is a way to represent relationships between things.

A graph contains:
- **Vertices (nodes):** the things being represented.
- **Edges:** the connections between those things.

In this lesson, you will represent a small network of connected services and visit reachable services using **Breadth-First Search (BFS)**.

> **Learning goal:** Understand nodes, edges, adjacency lists, visited sets, and a basic graph traversal.

## 1. What is a graph?

Imagine a small environment with services that communicate with one another:

~~~text
          api
         /   \
      auth   catalog
        |       |
      users   database
~~~

Each service is a **node**. Each line represents a connection, or **edge**.

Graphs are useful for modelling relationships such as:
- Services connected to other services.
- Network devices and their links.
- Dependencies between tasks.
- Routes between locations.
- Relationships between infrastructure resources.

These are examples of graph-shaped problems. A real service topology or dependency model needs accurate source data and may have direction, weights, and other rules not included in this exercise.

## 2. Directed and undirected graphs

Two common graph models are:

| Type | Meaning | Example |
| --- | --- | --- |
| Undirected | A connection can be followed in either direction | A simplified two-way network link |
| Directed | A connection has a direction | One task must run before another |

For this lesson, we use an **undirected graph**. If service A is connected to service B, the example records the connection in both directions.

## 3. Represent a graph with a Python dictionary

Python's standard collection types do not provide a dedicated general-purpose graph structure. A common beginner-friendly representation is an **adjacency list**.

An adjacency list stores each node and the nodes directly connected to it.

~~~python
network = {
    "api": ["auth", "catalog"],
    "auth": ["api", "users"],
    "catalog": ["api", "database"],
    "users": ["auth"],
    "database": ["catalog"],
}
~~~

Understand the structure:
- The dictionary key is a node.
- The list stored as its value contains its neighbors.
- For example, the neighbors of api are auth and catalog.
- Because this is an undirected example, the reverse connections are also present.

Read a node's neighbors:

~~~python
print(network["api"])
~~~

Output:

~~~text
['auth', 'catalog']
~~~

Check whether a node exists before looking it up:

~~~python
node = "api"

if node in network:
    print("Known node:", node)
else:
    print("Unknown node:", node)
~~~

Output:

~~~text
Known node: api
~~~

## 4. What is graph traversal?

**Traversal** means visiting nodes by following graph connections.

Two well-known traversal approaches are:

| Algorithm | Basic idea | Common structure |
| --- | --- | --- |
| Breadth-First Search (BFS) | Visit nearby nodes before moving farther away | Queue |
| Depth-First Search (DFS) | Follow one path farther before backtracking | Stack or recursion |

This lesson focuses on BFS. You have already learned queues in [11 Queues](../11%20Queues/README.md), so you can reuse that idea here.

## 5. Breadth-First Search (BFS)

BFS explores a graph level by level:

1. Start at a chosen node.
2. Mark it as visited.
3. Add it to a queue.
4. Remove the next node from the front of the queue.
5. Visit each neighbor that has not already been visited.
6. Mark each new neighbor as visited and add it to the queue.
7. Continue until the queue is empty.

The **visited** set is important. It prevents the traversal from visiting the same node repeatedly, especially when the graph contains cycles.

## 6. Implement BFS using Python

~~~python
from collections import deque


def breadth_first_search(graph, start):
    if start not in graph:
        return []

    visited = {start}
    pending = deque([start])
    visit_order = []

    while pending:
        current = pending.popleft()
        visit_order.append(current)

        for neighbor in graph[current]:
            if neighbor not in visited:
                visited.add(neighbor)
                pending.append(neighbor)

    return visit_order


network = {
    "api": ["auth", "catalog"],
    "auth": ["api", "users"],
    "catalog": ["api", "database"],
    "users": ["auth"],
    "database": ["catalog"],
}

print(breadth_first_search(network, "api"))
~~~

Output:

~~~text
['api', 'auth', 'catalog', 'users', 'database']
~~~

The order follows the neighbor order in the dictionary's lists. If you change the order of a node's neighbors, the traversal order may change too.

### Line-by-line explanation

1. Import deque so the traversal can use a FIFO queue.
2. If the starting node is not a key in the graph, return an empty list.
3. Put the starting node in visited immediately.
4. Create a queue containing the starting node.
5. Create an empty list to record the order in which nodes are visited.
6. Continue while the queue contains nodes.
7. Remove the oldest waiting node using popleft().
8. Add that node to the visit-order list.
9. Examine each neighbor of the current node.
10. If a neighbor has not been visited, mark it visited and add it to the queue.
11. Return the traversal order after the queue becomes empty.

Marking a node visited when it is added to the queue prevents the same node from being added multiple times before it is processed.

## 7. Dry run BFS

Start at api.

Initial state:

~~~text
Queue:   [api]
Visited: {api}
Order:   []
~~~

Then process each node:

| Step | Node removed | New neighbors added | Queue after step | Visit order |
| --- | --- | --- | --- | --- |
| 1 | api | auth, catalog | auth, catalog | api |
| 2 | auth | users | catalog, users | api, auth |
| 3 | catalog | database | users, database | api, auth, catalog |
| 4 | users | none | database | api, auth, catalog, users |
| 5 | database | none | empty | api, auth, catalog, users, database |

Final traversal:

~~~text
api → auth → catalog → users → database
~~~

The arrow shows the order in which BFS visits nodes. It does not mean that the graph's edges are directed.

## 8. Handle cycles and disconnected nodes

### Cycles

The example graph contains cycles. For instance, api connects to auth, and auth connects back to api.

The visited set prevents the traversal from repeatedly following that connection.

~~~python
assert breadth_first_search(network, "api") == [
    "api",
    "auth",
    "catalog",
    "users",
    "database",
]
~~~

### Disconnected nodes

A graph may contain nodes that cannot be reached from the chosen starting node.

~~~python
separate_network = {
    "api": ["auth"],
    "auth": ["api"],
    "worker": [],
}

print(breadth_first_search(separate_network, "api"))
~~~

Output:

~~~text
['api', 'auth']
~~~

The worker node is not visited because it is disconnected from api.

BFS from one starting node visits only the nodes reachable from that start. It does not automatically traverse every disconnected component.

### Unknown starting node

The function returns an empty list if the start node is not present as a graph key:

~~~python
assert breadth_first_search(network, "missing-service") == []
~~~

## 9. Test the implementation

Run these assertions after defining the function:

~~~python
simple_graph = {
    "a": ["b", "c"],
    "b": ["a", "d"],
    "c": ["a"],
    "d": ["b"],
}

assert breadth_first_search(simple_graph, "a") == ["a", "b", "c", "d"]
assert breadth_first_search(simple_graph, "c") == ["c", "a", "b", "d"]
assert breadth_first_search(simple_graph, "missing") == []

disconnected_graph = {
    "first": ["second"],
    "second": ["first"],
    "third": [],
}

assert breadth_first_search(disconnected_graph, "first") == [
    "first",
    "second",
]

print("Graph traversal tests passed.")
~~~

Expected output:

~~~text
Graph traversal tests passed.
~~~

## 10. Time and space complexity

Let:
- V be the number of vertices (nodes).
- E be the number of edges.

For an adjacency-list representation, BFS visits each reachable node at most once and examines the neighbor entries of visited nodes.

| Resource | Complexity | Reason |
| --- | --- | --- |
| Time | O(V + E) | Nodes and adjacency-list edges are examined |
| Extra space | O(V) | The visited set, queue, and visit-order list can hold nodes |

These are the standard complexity bounds for BFS with an adjacency-list representation. For an undirected graph, each edge is normally represented in both endpoints' neighbor lists.

## 11. Common beginner mistakes

1. **Forgetting the visited set:** Cycles can cause repeated visits or an endless traversal.
2. **Marking nodes visited too late:** Mark a neighbor visited when you enqueue it, not only when you remove it.
3. **Using a stack by accident:** BFS uses FIFO queue behavior; a stack produces depth-first behavior.
4. **Assuming all nodes are reachable:** A traversal from one start only visits its reachable component.
5. **Forgetting reverse links in an undirected graph:** If A connects to B, include B in A's neighbors and A in B's neighbors.
6. **Assuming traversal order is universal:** Neighbor ordering affects the order returned by this implementation.
7. **Treating the example as a real topology scanner:** The graph is supplied manually; the code does not discover services or inspect a network.

## 12. Beginner exercises

Try each exercise before opening its solution.

### Exercise 1: Read a node's neighbors

Given this graph, print the neighbors of web:

~~~python
graph = {
    "web": ["api", "cache"],
    "api": ["web"],
    "cache": ["web"],
}
~~~

<details>
<summary>Solution</summary>

~~~python
print(graph["web"])
~~~

Output:

~~~text
['api', 'cache']
~~~

</details>

### Exercise 2: Add a connection

Add an undirected connection between api and database in the graph below:

~~~python
graph = {
    "api": [],
    "database": [],
}
~~~

<details>
<summary>Solution</summary>

~~~python
graph["api"].append("database")
graph["database"].append("api")
~~~

Both sides are updated because the example connection is undirected.

</details>

### Exercise 3: Trace BFS

Use the BFS function to find the visit order starting from web:

~~~python
graph = {
    "web": ["api", "cache"],
    "api": ["web", "database"],
    "cache": ["web"],
    "database": ["api"],
}
~~~

<details>
<summary>Solution</summary>

~~~python
assert breadth_first_search(graph, "web") == [
    "web",
    "api",
    "cache",
    "database",
]
~~~

</details>

### Exercise 4: Find an unreachable node

Create a graph with two disconnected groups. Run BFS from a node in the first group and confirm that nodes in the second group are not returned.

<details>
<summary>Solution</summary>

~~~python
graph = {
    "a": ["b"],
    "b": ["a"],
    "x": ["y"],
    "y": ["x"],
}

assert breadth_first_search(graph, "a") == ["a", "b"]
~~~

</details>

### Exercise 5: Explain the visited set

In your own words, explain why BFS adds a neighbor to visited when it is added to the queue.

<details>
<summary>Suggested answer</summary>

A node may be connected to multiple already-visited nodes. Marking it visited when it is enqueued prevents it from being added to the queue repeatedly before it is processed.

</details>

## 13. Interview questions

1. What is a graph?
2. What is the difference between a vertex and an edge?
3. What is the difference between directed and undirected graphs?
4. What is an adjacency list?
5. What does BFS stand for?
6. Why does BFS use a queue?
7. Why does graph traversal need a visited set?
8. Does BFS from one node necessarily visit every node in a graph?
9. What is the time complexity of BFS with an adjacency list?
10. How does BFS differ from DFS?

## 14. Completion checklist

- [ ] I can explain nodes and edges.
- [ ] I can distinguish directed and undirected graphs.
- [ ] I can represent a small graph with a dictionary and lists.
- [ ] I can read a node's neighbors.
- [ ] I can explain BFS in simple words.
- [ ] I can use a queue to implement BFS.
- [ ] I understand why the visited set is necessary.
- [ ] I can identify disconnected nodes.
- [ ] I can dry-run BFS by hand.
- [ ] I can explain O(V + E) for adjacency-list BFS.

## 15. Official Python references

Python's documentation describes the language data structures used in this example. The graph representation and BFS algorithm are repository teaching examples, not built-in Python graph features.

- [Python Tutorial — Dictionaries](https://docs.python.org/3/tutorial/datastructures.html#dictionaries): key-value mappings used for the adjacency list.
- [Python Tutorial — More on Lists](https://docs.python.org/3/tutorial/datastructures.html#more-on-lists): lists used to store each node's neighbors.
- [Python Tutorial — Sets](https://docs.python.org/3/tutorial/datastructures.html#sets): set behavior used for tracking visited nodes.
- [Python Standard Library — collections.deque](https://docs.python.org/3/library/collections.html#collections.deque): queue operations used by BFS.

## Navigation

- Previous: [12 Sliding Window Basics](../12%20Sliding%20Window%20Basics/README.md)
- Also study: [11 Queues](../11%20Queues/README.md)
- [DSA for DevOps with Python — Roadmap](../README.md)

---

**Next step:** Draw a small graph on paper, write its adjacency list, and trace BFS from one starting node before running the code.
