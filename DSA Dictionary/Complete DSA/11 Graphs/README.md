# 11 — Graphs

Graphs are data structures for representing relationships between entities. They are one of the most important topics in DSA because many real-world systems are naturally modeled as networks.

> **Learning goal:** Understand graph terminology, representations, traversal, connectivity, cycle detection, shortest paths, topological ordering, weighted graphs, and the reasoning patterns used to solve graph problems.

---

## 1. What Is a Graph?

A graph consists of:

- **vertices** (also called nodes)
- **edges** connecting pairs of vertices

A simple example:

```text
A ---- B
|      |
|      |
C ---- D
```

Here:

```text
Vertices = {A, B, C, D}
Edges    = {A-B, A-C, B-D, C-D}
```

Graphs are useful when the important information is about **relationships between objects**, rather than a simple sequence or hierarchy.

Examples include:

- computer networks
- service dependencies
- road networks
- social relationships
- package dependencies
- workflow dependencies
- state transitions

These are examples of problems that can be modeled as graphs. They are not claims about the internal implementation of a particular external product.

---

## 2. Graph Terminology

### Vertex

A node in the graph.

### Edge

A relationship or connection between two vertices.

### Adjacent vertices

Two vertices connected directly by an edge.

### Degree

For an undirected graph, the degree of a vertex is the number of incident edges.

For directed graphs, we distinguish:

- **in-degree** — number of incoming edges
- **out-degree** — number of outgoing edges

### Path

A sequence of vertices connected by edges.

### Cycle

A path that returns to an earlier vertex according to the graph's cycle definition.

### Connected graph

In an undirected graph, every vertex is reachable from every other vertex.

### Component

A maximal connected portion of an undirected graph.

---

## 3. Directed vs Undirected Graphs

An **undirected graph** represents a two-way relationship:

```text
A ---- B
```

The edge connects `A` and `B` without a direction.

A **directed graph** gives the edge a direction:

```text
A ----> B
```

This means the relationship is from `A` to `B`.

Directed graphs are especially useful for dependency and workflow problems.

For example:

```text
Database ----> API
API ---------> Web
```

The arrow can represent a dependency direction chosen by the application model.

> **Repository design decision:** Always document what an edge means before implementing a graph algorithm. Reversing the meaning of an edge can produce a technically correct algorithm with an incorrect result for the intended problem.

---

## 4. Weighted vs Unweighted Graphs

In an **unweighted graph**, an edge has no numerical cost attached to it.

```text
A ---- B
```

In a **weighted graph**, an edge has a weight:

```text
A --5-- B
```

The weight might represent a cost, distance, latency, capacity, or another application-defined quantity.

The meaning of the weight must be established by the problem.

A shortest path in terms of number of edges is not necessarily the same as a minimum-cost path in a weighted graph.

---

## 5. Simple Graphs and Multigraphs

A **simple graph** generally has at most one edge between a given pair of vertices and does not contain self-loops under the usual definition.

A graph model may instead allow:

- parallel edges
- self-loops

Whether these are permitted is part of the graph definition for the problem.

Do not assume the input is a simple graph unless the problem or specification says so.

---

## 6. Graph Representation

The two foundational representations are:

1. adjacency list
2. adjacency matrix

The correct choice depends on the graph's size and density and on the operations required.

---

## 7. Adjacency List

An adjacency list stores the neighbors of each vertex.

For:

```text
A ---- B
|      |
C ---- D
```

an undirected representation could conceptually be:

```text
A: [B, C]
B: [A, D]
C: [A, D]
D: [B, C]
```

For a directed graph:

```text
A --> B
A --> C
```

could be represented as:

```text
A: [B, C]
B: []
C: []
```

Adjacency lists are generally efficient for sparse graphs because they store only existing edges.

A common Python representation is:

```python
from collections import defaultdict

graph = defaultdict(list)
graph["A"].append("B")
graph["B"].append("A")
```

The exact representation is a repository implementation choice.

---

## 8. Adjacency Matrix

An adjacency matrix uses a two-dimensional structure.

For vertices `A`, `B`, and `C`:

```text
      A B C
A     0 1 0
B     1 0 1
C     0 1 0
```

A `1` can indicate an edge and `0` no edge for an unweighted graph.

For weighted graphs, matrix entries can store weights, with a separate convention for the absence of an edge.

An adjacency matrix uses `O(V²)` storage for `V` vertices.

It can provide direct edge-existence lookup in `O(1)` when represented as a matrix.

---

## 9. Adjacency List vs Matrix

| Property | Adjacency List | Adjacency Matrix |
|---|---|---|
| Storage | `O(V + E)` | `O(V²)` |
| Neighbor iteration | Efficient for existing edges | Requires scanning row |
| Direct edge lookup | Depends on list structure | `O(1)` |
| Sparse graphs | Usually a good fit | Often wasteful |
| Dense graphs | Can still work | Often practical |

The exact complexity of a particular operation can depend on the representation and implementation details.

> **Rule:** Choose the representation based on the operations your algorithm actually needs.

---

## 10. Graph Traversal

The two foundational graph traversals are:

- **Breadth-First Search (BFS)**
- **Depth-First Search (DFS)**

Both can systematically explore reachable vertices.

The important difference is the order in which they explore the graph.

---

## 11. Breadth-First Search

BFS explores vertices level by level.

A queue is the natural data structure for BFS.

Conceptually:

```text
put starting vertex into queue
mark it visited

while queue is not empty:
    remove a vertex
    process it

    for each unvisited neighbor:
        mark neighbor visited
        add neighbor to queue
```

Example:

```text
        A
       / \
      B   C
     /     \
    D       E
```

Starting from `A`, one possible BFS order is:

```text
A B C D E
```

The exact order among neighbors can depend on the adjacency-list ordering.

---

## 12. Why Mark BFS Nodes When Enqueuing?

A common implementation detail is to mark a vertex visited when it is **enqueued**, rather than waiting until it is removed.

Consider:

```text
A ---- B
 \    /
  \  /
    C
```

Both `B` and `C` may discover the same neighbor.

Marking when enqueued prevents the same vertex from being inserted into the queue repeatedly.

This is an implementation detail that helps maintain the intended traversal behavior and efficiency.

---

## 13. BFS and Shortest Paths in Unweighted Graphs

In an unweighted graph, BFS explores vertices by increasing number of edges from the starting vertex.

Therefore, when the graph is modeled appropriately, the first BFS discovery of a reachable vertex gives a shortest path measured by **number of edges**.

Example:

```text
A -- B -- D
|         |
C --------
```

From `A`, BFS can discover vertices by distance:

```text
Distance 0: A
Distance 1: B, C
Distance 2: D
```

This property makes BFS useful for unweighted shortest-path problems.

It does **not** mean BFS solves arbitrary weighted shortest-path problems.

---

## 14. BFS Distance and Parent Tracking

To reconstruct a shortest path, maintain two pieces of information:

```text
distance[vertex]
parent[vertex]
```

When a new vertex is discovered from `current`:

```text
distance[neighbor] = distance[current] + 1
parent[neighbor] = current
```

After reaching the target, follow parent pointers backward:

```text
target → parent → parent → ... → source
```

Then reverse the sequence.

This separates two concerns:

```text
finding the distance
```

from:

```text
reconstructing the actual path
```

---

## 15. Depth-First Search

DFS explores as deeply as possible before backtracking.

It can be implemented using:

- recursion
- an explicit stack

Recursive conceptual form:

```python
def dfs(node):
    if node in visited:
        return

    visited.add(node)

    for neighbor in graph[node]:
        dfs(neighbor)
```

The exact implementation must account for the graph representation and whether recursion depth is appropriate for the input size.

---

## 16. DFS with an Explicit Stack

DFS can also be implemented iteratively:

```text
push starting vertex

while stack is not empty:
    pop a vertex

    if already visited:
        continue

    mark visited
    process vertex

    push its neighbors
```

Neighbor insertion order can affect the traversal order.

This is an important connection to the previous chapter on stacks.

---

## 17. BFS vs DFS

| Question | BFS | DFS |
|---|---|---|
| Main structure | Queue | Stack / recursion |
| Exploration | Level by level | Deep before backtracking |
| Unweighted shortest path | Natural choice | Not generally shortest |
| Cycle detection | Possible | Possible |
| Components | Yes | Yes |
| Path exploration | Possible | Natural fit |

Neither traversal is universally superior.

Choose based on the property the problem requires.

---

## 18. Complexity of BFS and DFS

With an adjacency-list representation, a standard BFS or DFS that visits each reachable vertex and examines each relevant adjacency entry runs in:

```text
O(V + E)
```

where:

- `V` = number of vertices
- `E` = number of edges

The exact space usage includes the visited structure and traversal data structure.

For a graph that is disconnected, one traversal starting from a single vertex may not reach every vertex.

---

## 19. Traversing a Disconnected Graph

Consider:

```text
A -- B       C -- D
```

Starting BFS from `A` reaches only:

```text
A, B
```

It does not automatically reach `C` or `D`.

To process the entire graph:

```text
for every vertex:
    if not visited:
        start a new BFS or DFS
```

This produces one traversal per connected component.

The overall adjacency-list complexity remains `O(V + E)` when each vertex and edge is processed a constant number of times.

---

## 20. Connected Components

For an undirected graph, connected components can be found using repeated DFS or BFS.

Conceptually:

```text
components = 0

for vertex in graph:
    if vertex is not visited:
        components += 1
        traverse from vertex
```

Every new traversal begins a new component.

This is a common pattern:

> **Outer loop over all vertices + inner graph traversal = process every component.**

---

## 21. Cycle Detection in an Undirected Graph

A cycle exists when traversal can reach an already explored vertex through an edge that is not simply the edge back to the current vertex's parent.

During DFS, conceptually track:

```text
current vertex
parent vertex
```

If a visited neighbor is not the parent, that can indicate a cycle in the usual undirected simple-graph model.

The exact logic must match the graph model, especially if self-loops or parallel edges are allowed.

---

## 22. Cycle Detection in a Directed Graph

Directed graphs require different reasoning.

A common DFS technique tracks three states:

```text
0 = unvisited
1 = currently being explored
2 = completely explored
```

If DFS encounters an edge to a vertex currently in state `1`, it has found a back edge to the active recursion path, indicating a directed cycle.

This technique is especially useful for dependency graphs.

---

## 23. Topological Sorting

A topological ordering is an ordering of vertices in a directed acyclic graph (DAG) such that for every directed edge:

```text
A → B
```

`A` appears before `B` in the ordering.

Example:

```text
Build source
     ↓
Compile
     ↓
Test
     ↓
Deploy
```

A valid ordering is:

```text
Build source, Compile, Test, Deploy
```

A topological ordering exists only for a directed acyclic graph under the standard definition.

---

## 24. Kahn's Algorithm

One common topological-sort approach repeatedly processes vertices with zero in-degree.

Conceptually:

```text
calculate in-degree of every vertex
put all zero-in-degree vertices into a queue

while queue is not empty:
    remove a vertex
    output it

    for each outgoing edge:
        decrease neighbor's in-degree

        if neighbor's in-degree becomes zero:
            add neighbor to queue
```

If fewer than `V` vertices are output, the graph contains a cycle under the standard DAG model.

With adjacency lists, the algorithm runs in `O(V + E)` time.

---

## 25. DFS-Based Topological Ordering

A topological order can also be constructed with DFS by placing a vertex after all of its outgoing neighbors have been processed and then reversing the resulting completion order.

For directed graphs, cycle detection must be handled as well.

The key reasoning pattern is:

```text
finish dependencies first
then place the dependent vertex
```

This connects topological sorting with postorder traversal.

---

## 26. Weighted Graphs

Weighted graphs attach a numerical value to each edge.

Example:

```text
A --4-- B --2-- C
 \             /
  -----7-------
```

The graph's weight could represent distance, cost, latency, or another problem-specific quantity.

Once weights are introduced, the phrase **shortest path** normally means minimum total weight, not minimum number of edges.

---

## 27. Dijkstra's Algorithm

Dijkstra's algorithm solves a class of single-source shortest-path problems for graphs whose edge weights satisfy the algorithm's required non-negative-weight condition.

The high-level idea is:

```text
start with source distance = 0
all other distances = infinity

repeatedly select the unsettled vertex with smallest tentative distance
relax its outgoing edges
```

A priority queue can make the repeated minimum selection efficient.

The details of the exact complexity depend on the graph representation and priority-queue implementation.

> **Important:** Do not apply Dijkstra blindly to graphs with negative edge weights. The algorithm's correctness assumptions matter.

---

## 28. Edge Relaxation

Relaxation is the fundamental operation behind many shortest-path algorithms.

Suppose:

```text
current distance = d[u]
edge weight = w
known distance to v = d[v]
```

The candidate distance is:

```text
d[u] + w
```

If the candidate is smaller:

```text
d[v] = d[u] + w
parent[v] = u
```

This operation is called **relaxing** the edge.

Understanding relaxation is more important than memorizing a specific shortest-path implementation.

---

## 29. Bellman-Ford Preview

Bellman-Ford is another shortest-path algorithm with different capabilities and trade-offs from Dijkstra's algorithm.

It can handle graphs with negative edge weights under its standard assumptions and can be used to detect reachable negative-weight cycles.

Its typical time complexity is:

```text
O(VE)
```

This chapter introduces the concept; detailed implementation belongs in the advanced graph algorithms section.

---

## 30. Minimum Spanning Tree Preview

A **minimum spanning tree (MST)** is a spanning tree of a connected, weighted, undirected graph with minimum total edge weight.

Two classic algorithm families are:

- Kruskal's algorithm
- Prim's algorithm

They solve a different problem from shortest paths.

Shortest path asks about reaching one vertex from another with minimum path cost.

MST asks for a minimum-weight structure connecting all vertices.

Do not confuse these objectives.

---

## 31. Union-Find / Disjoint Set Union

The disjoint-set data structure maintains a collection of disjoint sets and supports operations commonly called:

- `find`
- `union`

It is useful for connectivity problems and is a key component of Kruskal's algorithm.

With path compression and union by rank or size, the amortized complexity is commonly expressed using the inverse Ackermann function, which grows extremely slowly.

This data structure will be studied more deeply with advanced graph algorithms and data structures.

---

## 32. Graph Problem Patterns

Many graph interview problems fall into recognizable patterns.

### Pattern 1 — Reachability

Question:

> Can I get from `A` to `B`?

Use BFS or DFS.

### Pattern 2 — Shortest path, unweighted

Question:

> What is the minimum number of edges from `A` to `B`?

Use BFS.

### Pattern 3 — Dependency ordering

Question:

> In what order can these tasks be completed?

Think DAG + topological sorting.

### Pattern 4 — Detect a cycle

Question:

> Does the dependency or relationship structure contain a cycle?

Think carefully about directed vs undirected cycle detection.

### Pattern 5 — Components

Question:

> How many independent groups exist?

Use repeated BFS/DFS or an appropriate connectivity structure.

### Pattern 6 — Minimum weighted path

Question:

> What is the minimum total cost from source to destination?

Choose a shortest-path algorithm based on the edge-weight assumptions.

---

## 33. Graph Invariants

Graph algorithms become easier when the invariant is written down before coding.

Examples:

### BFS invariant

When a vertex is first discovered in an unweighted graph, its recorded distance is the minimum number of edges from the source under the standard BFS model.

### DFS invariant

A visited vertex has already been encountered by the traversal, so it should not be recursively explored again merely because another edge reaches it.

### Topological-sort invariant

A vertex is ready to process when all of its required incoming dependencies have been satisfied, represented by zero remaining in-degree in Kahn's algorithm.

### Dijkstra invariant

Under its required non-negative-weight assumptions, selecting the smallest tentative distance gives a vertex whose shortest-path distance can be finalized in the standard algorithm.

Writing these invariants makes debugging much easier.

---

## 34. Common Graph Pitfalls

### Mistake 1 — Forgetting disconnected components

A single BFS or DFS does not necessarily visit every vertex.

### Mistake 2 — Marking visited too late in BFS

This can cause repeated queue insertions.

### Mistake 3 — Using BFS for weighted shortest paths without justification

BFS minimizes edge count, not arbitrary total weight.

### Mistake 4 — Using Dijkstra with unsupported negative weights

Check the algorithm's assumptions first.

### Mistake 5 — Mixing directed and undirected logic

Cycle detection and degree calculations differ.

### Mistake 6 — Reversing dependency direction

Clearly define what `A → B` means.

### Mistake 7 — Assuming traversal order is unique

Neighbor ordering can change BFS/DFS output without changing reachability.

### Mistake 8 — Confusing shortest path with minimum spanning tree

They optimize different objectives.

---

## 35. Edge Cases to Test

Always test graph algorithms with:

- empty graph
- one vertex
- one edge
- disconnected graph
- isolated vertex
- self-loop when allowed
- duplicate/parallel edges when allowed
- cycle
- no cycle
- directed graph
- undirected graph
- dense graph
- sparse graph
- multiple shortest paths
- unreachable destination
- zero-weight edges where allowed
- negative edges only when the chosen algorithm supports them

Testing these cases often reveals incorrect assumptions about the graph model.

---

## 36. A Repeatable Graph Problem-Solving Process

When you see a graph problem, use this sequence:

### Step 1 — Identify the entities

What are the vertices?

### Step 2 — Identify the relationship

What does an edge mean?

### Step 3 — Determine direction

Is `A → B` different from `B → A`?

### Step 4 — Determine weights

Does each edge have a cost?

### Step 5 — Determine the graph constraints

Ask about:

```text
V = number of vertices
E = number of edges
```

Also check whether the graph is sparse, dense, connected, cyclic, or acyclic.

### Step 6 — Choose the representation

Usually adjacency list for sparse graph traversal; matrix when direct edge lookup or dense structure makes it appropriate.

### Step 7 — Identify the required property

```text
reachability → BFS / DFS
unweighted shortest path → BFS
dependency ordering → topological sort
weighted shortest path → appropriate shortest-path algorithm
connectivity → BFS / DFS / DSU
```

### Step 8 — Write the invariant

State what `visited`, `distance`, `parent`, `in_degree`, or other state means.

### Step 9 — Trace a tiny example

Use a graph with only a few vertices.

### Step 10 — Test adversarial cases

Especially cycles, disconnected components, duplicate edges, and unreachable targets.

---

## 37. Practice Progression

### Level 1 — Fundamentals

1. Build an adjacency list.
2. Count vertices and edges.
3. List neighbors of a vertex.
4. Implement BFS.
5. Implement DFS.

### Level 2 — Traversal Problems

6. Find whether a path exists.
7. Count connected components.
8. Find the size of each component.
9. Find shortest distance in an unweighted graph.
10. Reconstruct a shortest path.

### Level 3 — Structural Problems

11. Detect an undirected cycle.
12. Detect a directed cycle.
13. Determine whether a graph is bipartite.
14. Produce a topological ordering.
15. Detect impossible dependency ordering.

### Level 4 — Weighted Graphs

16. Implement weighted adjacency lists.
17. Implement edge relaxation.
18. Solve non-negative weighted shortest-path problems.
19. Compare shortest-path algorithms conceptually.
20. Study minimum spanning tree algorithms.

### Level 5 — Advanced

21. Union-Find / DSU.
22. Kruskal's algorithm.
23. Prim's algorithm.
24. Bellman-Ford.
25. All-pairs shortest paths.
26. Strongly connected components.
27. Advanced graph traversal patterns.

---

## 38. DevOps-Oriented Mini Project — Service Dependency Analyzer

### Goal

Build a small program that models services as a directed graph and answers questions such as:

- Which services are reachable from a given service?
- Is there a dependency cycle?
- What is a valid deployment order when dependencies form a DAG?
- Which services have no incoming dependencies?
- Which services become affected when a dependency changes?

### Suggested model

```text
database → api
api      → worker
api      → frontend
worker   → queue
```

### Suggested workflow

```text
parse service relationships
        ↓
build adjacency list
        ↓
run graph analysis
        ↓
report reachability / cycles / ordering
```

This is a **repository demonstration project**, not a claim about how any specific DevOps platform models dependencies internally.

### Extension ideas

- export the graph as text
- calculate dependency depth
- identify highly connected services
- detect circular dependencies
- generate a topological deployment order when possible
- add test cases for disconnected services

---

## 39. Interview Questions

Be able to explain these without memorizing a script:

1. What is the difference between a tree and a general graph?
2. What is the difference between directed and undirected graphs?
3. When would you use an adjacency list instead of an adjacency matrix?
4. Why does BFS use a queue?
5. Why does DFS use a stack or recursion?
6. Why does BFS find shortest paths in an unweighted graph?
7. Why does one BFS not necessarily visit a disconnected graph completely?
8. How do you detect a cycle in an undirected graph?
9. How is directed cycle detection different?
10. What is a topological ordering?
11. Why must a graph for topological sorting be acyclic?
12. What is edge relaxation?
13. When is Dijkstra's algorithm appropriate?
14. Why can't Dijkstra's algorithm be applied blindly to negative edge weights?
15. What is the difference between shortest path and minimum spanning tree?
16. What are connected components?
17. What is the typical adjacency-list complexity of BFS or DFS?
18. How would you model service dependencies as a graph?

---

## 40. Completion Checklist

Before moving on, you should be able to:

- [ ] Define vertices and edges.
- [ ] Explain directed vs undirected graphs.
- [ ] Explain weighted vs unweighted graphs.
- [ ] Build an adjacency list.
- [ ] Build an adjacency matrix.
- [ ] Implement BFS.
- [ ] Implement DFS.
- [ ] Explain BFS shortest paths in unweighted graphs.
- [ ] Handle disconnected graphs.
- [ ] Find connected components.
- [ ] Detect cycles.
- [ ] Explain topological sorting.
- [ ] Explain edge relaxation.
- [ ] State the assumptions behind Dijkstra's algorithm.
- [ ] Distinguish shortest paths from minimum spanning trees.
- [ ] Analyze graph algorithms using `V` and `E`.
- [ ] Model a practical dependency problem as a graph.

---

## 41. Sources and Quality Notes

This chapter follows standard graph terminology and algorithmic definitions. Python-specific implementation guidance should be checked against the official Python documentation.

### Python sources

- Python `collections` documentation: https://docs.python.org/3/library/collections.html
- Python `heapq` documentation: https://docs.python.org/3/library/heapq.html

### Quality policy

- Complexity statements assume the representation and operation model described in the relevant section.
- Traversal order can depend on neighbor ordering.
- Graph behavior involving duplicates, self-loops, parallel edges, or negative weights depends on the stated graph model and algorithm assumptions.
- Repository examples are learning demonstrations, not claims about the internal implementation of external systems.

---

## Next

Continue to:

**[12 — Greedy Algorithms](../12%20Greedy%20Algorithms/README.md)**

[← Previous: 10 — Heaps and Priority Queues](../10%20Heaps%20and%20Priority%20Queues/README.md)  
[↑ Complete DSA](../README.md)
