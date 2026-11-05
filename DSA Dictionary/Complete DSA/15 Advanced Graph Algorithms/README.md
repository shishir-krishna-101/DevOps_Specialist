# 15 — Advanced Graph Algorithms

This chapter builds on the graph fundamentals introduced earlier. It focuses on algorithms for shortest paths, spanning structures, connectivity, and graph ordering.

Advanced graph problems often depend on details that are easy to overlook: whether edges are directed, whether weights can be negative, whether the graph is connected, and whether the task asks for one path, all-pairs distances, or a structural property.

Always define the graph model and algorithm assumptions before choosing an implementation.

---

## Learning objectives

By the end of this chapter, you should be able to:

- Choose between BFS, Dijkstra, Bellman–Ford, and Floyd–Warshall for shortest-path problems.
- Explain why Dijkstra requires non-negative edge weights.
- Detect negative-weight cycles that are reachable from a source.
- Calculate shortest paths between every pair of vertices.
- Explain minimum spanning trees and compare Prim's and Kruskal's algorithms.
- Use disjoint-set union (DSU) to track connected components.
- Understand strongly connected components in directed graphs.
- Recognize bridges and articulation points in undirected graphs.
- Analyze graph algorithms in terms of vertices and edges.

## 1. Model the graph first

Before selecting an algorithm, record:

- **Vertices (V):** the objects or states.
- **Edges (E):** the relationships or transitions.
- **Direction:** directed or undirected.
- **Weights:** unweighted, non-negative, or possibly negative.
- **Connectivity:** connected, disconnected, or reachable only from a selected source.
- **Question:** one path, shortest path, all-pairs shortest paths, minimum spanning tree, or connectivity structure.

The graph representation also matters. An adjacency list is generally convenient for sparse graphs; an adjacency matrix provides direct edge lookup and is often used by dense-graph or all-pairs algorithms.

For a weighted adjacency list, a common representation is:

```python
graph = {
    "A": [("B", 4), ("C", 2)],
    "B": [("A", 4), ("C", 1)],
    "C": [("A", 2), ("B", 1)],
}
```

Each pair contains a neighboring vertex and the edge weight.

## 2. Shortest-path algorithm selection

The correct shortest-path algorithm depends on graph properties and the required output.

| Algorithm | Graph assumptions | Typical purpose |
|---|---|---|
| BFS | Unweighted edges (or equal edge costs) | Single-source shortest path by edge count |
| Dijkstra | Non-negative edge weights | Single-source shortest paths |
| Bellman–Ford | Weighted directed edges; negative edges allowed | Single-source distances and reachable negative-cycle detection |
| Floyd–Warshall | Weighted graph represented as a matrix; negative edges allowed | All-pairs shortest paths and negative-cycle indication |

These algorithms are not interchangeable. In particular, Dijkstra's standard correctness guarantee does not apply to graphs with negative-weight edges.

## 3. Dijkstra's algorithm

Dijkstra's algorithm calculates shortest distances from one source vertex when all edge weights are non-negative.

### Core idea

1. Set the source distance to zero.
2. Set other known distances to infinity.
3. Use a min-priority queue to select the unsettled vertex with the smallest tentative distance.
4. For each outgoing edge, attempt to improve the neighbor's distance.
5. If a distance improves, update it and add the new candidate to the priority queue.
6. Continue until the queue is empty or the required destination has been settled.

The update operation is called **relaxation**.

For an edge from u to v with weight w:

```text
candidate = distance[u] + w

if candidate < distance[v]:
    distance[v] = candidate
```

### Python implementation

This implementation uses a min-heap. It may contain older entries for a vertex; those entries are skipped when removed if they no longer match the best known distance.

```python
import heapq
from math import inf


def dijkstra(graph, source):
    """Return shortest distances from source in a graph
    with non-negative edge weights.

    graph: {vertex: [(neighbor, weight), ...]}
    """
    if source not in graph:
        raise ValueError("source must be a graph vertex")

    for neighbors in graph.values():
        for _, weight in neighbors:
            if weight < 0:
                raise ValueError("Dijkstra requires non-negative weights")

    distances = {vertex: inf for vertex in graph}
    distances[source] = 0

    priority_queue = [(0, source)]

    while priority_queue:
        current_distance, vertex = heapq.heappop(priority_queue)

        if current_distance != distances[vertex]:
            continue

        for neighbor, weight in graph[vertex]:
            candidate = current_distance + weight

            if candidate < distances[neighbor]:
                distances[neighbor] = candidate
                heapq.heappush(
                    priority_queue,
                    (candidate, neighbor),
                )

    return distances


weighted_graph = {
    "A": [("B", 4), ("C", 2)],
    "B": [("A", 4), ("C", 1), ("D", 5)],
    "C": [("A", 2), ("B", 1), ("D", 8)],
    "D": [("B", 5), ("C", 8)],
}

print(dijkstra(weighted_graph, "A"))
```

### Complexity

With an adjacency list and a binary heap, a common bound for this implementation is:

- Time: O((V + E) log V), with the usual heap-operation accounting.
- Space: O(V) for distances and queue storage, excluding the graph itself; lazy duplicate queue entries can make the queue hold O(E) entries over the run.

Exact bounds depend on the priority-queue implementation and graph representation.

### Why negative weights are a problem

Dijkstra finalizes vertices in increasing tentative-distance order. A negative edge can later create a shorter path to a vertex that was already treated as settled. This invalidates the algorithm's key reasoning.

Use Bellman–Ford when negative edge weights are permitted and single-source shortest paths are required.

## 4. Reconstructing a shortest path

Distances alone do not show which route produced them. Store a predecessor whenever a relaxation improves a distance.

```python
import heapq
from math import inf


def dijkstra_with_paths(graph, source):
    if source not in graph:
        raise ValueError("source must be a graph vertex")

    for neighbors in graph.values():
        for _, weight in neighbors:
            if weight < 0:
                raise ValueError("Dijkstra requires non-negative weights")

    distances = {vertex: inf for vertex in graph}
    previous = {vertex: None for vertex in graph}
    distances[source] = 0

    queue = [(0, source)]

    while queue:
        current_distance, vertex = heapq.heappop(queue)

        if current_distance != distances[vertex]:
            continue

        for neighbor, weight in graph[vertex]:
            candidate = current_distance + weight

            if candidate < distances[neighbor]:
                distances[neighbor] = candidate
                previous[neighbor] = vertex
                heapq.heappush(queue, (candidate, neighbor))

    return distances, previous


def build_path(previous, source, target):
    if target not in previous:
        raise ValueError("target is not in the predecessor map")

    path = []
    current = target

    while current is not None:
        path.append(current)
        if current == source:
            break
        current = previous[current]

    if not path or path[-1] != source:
        return []

    return list(reversed(path))
```

The path reconstruction walks backwards through predecessor links. If the target is unreachable, the predecessor chain does not reach the source and the function returns an empty list.

The graph's vertices must be hashable dictionary keys. This example assumes every neighbor referenced by an edge is also present as a graph key.

## 5. Bellman–Ford algorithm

Bellman–Ford calculates single-source shortest paths when edge weights may be negative. It can also detect a negative-weight cycle reachable from the source.

### Core idea

For a graph with V vertices:

1. Initialize the source distance to zero and all other distances to infinity.
2. Relax every edge repeatedly, for at most V - 1 passes.
3. Stop early if a complete pass makes no change.
4. Make one additional pass. If any reachable edge can still be relaxed, a reachable negative-weight cycle exists.

Why V - 1 passes? If no relevant negative cycle exists, a shortest simple path can contain at most V - 1 edges.

### Python implementation

Represent edges as (source, destination, weight) tuples.

```python
from math import inf


def bellman_ford(vertices, edges, source):
    """Return (distances, has_reachable_negative_cycle)."""
    if source not in vertices:
        raise ValueError("source must be a graph vertex")

    distances = {vertex: inf for vertex in vertices}
    distances[source] = 0

    for _ in range(len(vertices) - 1):
        changed = False

        for start, end, weight in edges:
            if distances[start] == inf:
                continue

            candidate = distances[start] + weight
            if candidate < distances[end]:
                distances[end] = candidate
                changed = True

        if not changed:
            break

    for start, end, weight in edges:
        if distances[start] == inf:
            continue

        if distances[start] + weight < distances[end]:
            return distances, True

    return distances, False


vertices = ["A", "B", "C", "D"]
edges = [
    ("A", "B", 4),
    ("A", "C", 5),
    ("B", "C", -2),
    ("C", "D", 3),
]

print(bellman_ford(vertices, edges, "A"))
```

### Complexity

- Time: O(V × E).
- Space: O(V) for the distance map, excluding the edge list.

The negative-cycle check detects a negative cycle reachable from the chosen source. A negative cycle in a disconnected component is not detected by this single-source procedure.

If a reachable negative cycle exists, shortest-path distances for vertices affected by that cycle are not finite well-defined minima: repeatedly traversing the cycle can reduce path cost without bound.

## 6. Floyd–Warshall: all-pairs shortest paths

Floyd–Warshall computes shortest-path distances between every pair of vertices.

It uses dynamic programming over allowed intermediate vertices.

### State interpretation

Let dist[i][j] be the best known distance from vertex i to vertex j.

When considering vertex k as an additional allowed intermediate vertex:

```text
dist[i][j] = min(
    dist[i][j],
    dist[i][k] + dist[k][j]
)
```

The algorithm checks whether routing through k improves the current path from i to j.

### Initialization

- Distance from a vertex to itself is zero.
- A direct edge initializes its edge weight.
- A missing edge is infinity.
- If multiple direct edges exist, keep the minimum edge weight for the distance matrix.

### Python implementation

```python
from math import inf


def floyd_warshall(matrix):
    """Return all-pairs shortest distances.

    matrix[i][j] is the direct edge weight, or inf if
    there is no direct edge. Diagonal entries should be 0.
    """
    size = len(matrix)

    if any(len(row) != size for row in matrix):
        raise ValueError("matrix must be square")

    distances = [row[:] for row in matrix]

    for intermediate in range(size):
        for start in range(size):
            if distances[start][intermediate] == inf:
                continue

            for end in range(size):
                if distances[intermediate][end] == inf:
                    continue

                candidate = (
                    distances[start][intermediate]
                    + distances[intermediate][end]
                )

                if candidate < distances[start][end]:
                    distances[start][end] = candidate

    has_negative_cycle = any(
        distances[index][index] < 0
        for index in range(size)
    )

    return distances, has_negative_cycle
```

### Complexity

For V vertices:

- Time: O(V³).
- Space: O(V²) for the distance matrix.

Floyd–Warshall supports negative edge weights, but a negative cycle makes shortest-path distances undefined for affected pairs. The returned flag indicates whether a negative cycle is present in the matrix's graph.

## 7. Shortest-path comparison

| Requirement | Suitable starting point |
|---|---|
| Unweighted shortest path from one source | BFS |
| Non-negative weighted shortest path from one source | Dijkstra |
| Negative edges allowed, single source | Bellman–Ford |
| Distances for every vertex pair | Floyd–Warshall, if its O(V³) cost is appropriate |

Always check whether the graph is directed or undirected and whether all relevant vertices are represented.

## 8. Minimum spanning trees

A **spanning tree** of a connected, undirected graph:

- Includes every vertex.
- Is connected.
- Contains no cycles.
- Has exactly V - 1 edges.

A **minimum spanning tree (MST)** is a spanning tree with minimum possible total edge weight.

If the graph is disconnected, a spanning tree covering every vertex does not exist. A minimum spanning forest consists of a minimum spanning tree for each connected component.

An MST is not the same as a shortest-path tree. An MST minimizes the total weight of selected edges; a shortest-path tree preserves shortest-path distances from a chosen source.

## 9. Kruskal's algorithm

Kruskal's algorithm builds an MST by considering edges in non-decreasing weight order.

### Steps

1. Sort all edges by weight.
2. Start with every vertex in its own component.
3. Consider edges from smallest to largest.
4. Add an edge if its endpoints are currently in different components.
5. Merge those components.
6. Stop after selecting V - 1 edges, if the graph is connected.

A disjoint-set union structure supports the component checks and merges.

### Disjoint-set union (DSU)

DSU maintains a partition of elements into disjoint sets.

Two common operations:

- **Find:** identify the representative of the set containing an element.
- **Union:** merge the sets containing two elements.

Path compression and union by size/rank make operations very efficient in practice; with both standard optimizations, the amortized complexity per operation is commonly expressed using the inverse Ackermann function.

### Python implementation

```python
class DisjointSet:
    def __init__(self, vertices):
        self.parent = {vertex: vertex for vertex in vertices}
        self.size = {vertex: 1 for vertex in vertices}

    def find(self, vertex):
        if vertex not in self.parent:
            raise KeyError(vertex)

        if self.parent[vertex] != vertex:
            self.parent[vertex] = self.find(self.parent[vertex])

        return self.parent[vertex]

    def union(self, first, second):
        root_first = self.find(first)
        root_second = self.find(second)

        if root_first == root_second:
            return False

        if self.size[root_first] < self.size[root_second]:
            root_first, root_second = root_second, root_first

        self.parent[root_second] = root_first
        self.size[root_first] += self.size[root_second]
        return True


def kruskal(vertices, edges):
    dsu = DisjointSet(vertices)
    selected = []
    total_weight = 0

    for weight, first, second in sorted(edges):
        if dsu.union(first, second):
            selected.append((first, second, weight))
            total_weight += weight

            if len(selected) == len(vertices) - 1:
                break

    if len(vertices) > 0 and len(selected) != len(vertices) - 1:
        return None, None

    return selected, total_weight
```

The edge format for this function is (weight, first_vertex, second_vertex). It returns (None, None) if a non-empty graph is disconnected and no spanning tree can be formed. For an empty vertex collection, it returns an empty edge list and total weight zero.

### Complexity

- Sorting edges: O(E log E).
- DSU operations: near-constant amortized time per operation with path compression and union by size.
- Total: O(E log E), dominated by sorting in the usual comparison model.
- Additional DSU storage: O(V).

## 10. Prim's algorithm

Prim's algorithm grows one tree from a starting vertex.

### Steps

1. Choose a starting vertex.
2. Add its outgoing edges to a min-priority queue.
3. Select the minimum-weight edge that reaches a vertex not already in the tree.
4. Add that vertex and its outgoing candidate edges.
5. Repeat until every vertex in the connected component has been reached.

A heap can efficiently choose the next minimum candidate edge.

Prim and Kruskal solve the MST problem using different growth strategies. Prim grows a connected tree; Kruskal grows a forest and merges components.

For a disconnected graph, running Prim from one starting vertex covers only that connected component. To produce a minimum spanning forest, repeat from each unvisited vertex.

## 11. Strongly connected components

In a directed graph, a strongly connected component (SCC) is a maximal set of vertices where every vertex can reach every other vertex in the set.

SCCs help compress a directed graph into a component graph. If each SCC is replaced by one node, the resulting condensation graph is a directed acyclic graph.

Two well-known algorithms are:

- **Kosaraju's algorithm:** uses DFS finishing order and a traversal of the transposed graph.
- **Tarjan's algorithm:** uses DFS discovery indices, low-link values, and a stack.

Both run in O(V + E) time with an adjacency-list representation.

### Kosaraju's algorithm: conceptual steps

1. Run DFS on the original graph and record vertices by finishing time.
2. Reverse every edge to construct the transposed graph.
3. Process vertices in decreasing finishing-time order.
4. Run DFS on the transposed graph from each unassigned vertex.
5. Each traversal identifies one SCC.

This section introduces the structure. An implementation should use a clearly defined visited set, finishing-order list, and reversed adjacency list. For very deep graphs, consider iterative DFS to avoid relying on the language's recursion depth.

## 12. Bridges and articulation points

These are connectivity concepts in undirected graphs.

### Bridge

A bridge is an edge whose removal increases the number of connected components.

### Articulation point

An articulation point is a vertex whose removal, together with its incident edges, increases the number of connected components.

DFS-based algorithms identify these using discovery times and low-link values.

- A DFS tree edge can be a bridge when the child subtree cannot reach the parent or an ancestor through a back edge.
- A non-root DFS vertex is an articulation point when a child subtree cannot reach an ancestor above that vertex.
- The DFS root has a separate rule: it is an articulation point when it has more than one DFS-tree child.

These rules assume an undirected graph and require careful handling of parent edges and disconnected components.

## 13. Topological ordering and DAG shortest paths

A directed acyclic graph (DAG) has no directed cycles. A topological ordering places every vertex before all vertices it points to.

Once a topological order is available, shortest paths in a DAG can be computed by processing vertices in that order and relaxing outgoing edges.

Unlike Dijkstra, this DAG shortest-path approach can handle negative edge weights because the acyclic order prevents a path from returning to an earlier vertex.

This is useful when dependencies naturally form a DAG, such as build steps or prerequisite relationships. If cycles exist, a topological ordering does not exist for the entire directed graph.

## 14. Common mistakes

- Using Dijkstra when negative edge weights are present.
- Forgetting that Bellman–Ford's negative-cycle check is source-reachability dependent.
- Treating an unreachable vertex's distance as a finite number.
- Confusing shortest-path trees with minimum spanning trees.
- Running Prim once on a disconnected graph and claiming the result spans all vertices.
- Forgetting to reject cycle-forming edges in Kruskal.
- Mixing up the edge tuple order expected by an implementation.
- Failing to include isolated vertices in the graph representation.
- Treating directed connectivity as undirected connectivity.
- Applying bridge or articulation-point rules to directed graphs.
- Ignoring recursion-depth risks in recursive DFS implementations.
- Using infinity arithmetic without checking whether a path segment is reachable.

## 15. Testing and validation

Create small graphs where the expected answer can be worked out by hand.

### Shortest paths

- Single vertex and no edges.
- Disconnected vertices.
- Multiple paths with different weights.
- Zero-weight edges.
- Negative edge without a negative cycle for Bellman–Ford.
- Reachable negative cycle.
- Negative cycle in a component unreachable from the source.
- Directed edges where reverse travel is not possible.

### Minimum spanning trees

- A connected graph with a unique MST.
- A graph with multiple valid MSTs of equal total weight.
- Equal-weight edges.
- A cycle.
- A disconnected graph.
- A single vertex.
- An empty graph, if supported by the interface.

### Connectivity structures

- A directed graph with one SCC.
- A directed graph with several SCCs.
- An undirected chain with bridges.
- A cycle with no bridges.
- A graph with an articulation point.
- A disconnected undirected graph.

For small weighted graphs, compare shortest-path implementations against a simpler trusted reference or manually enumerated paths. Validate MST output by checking that it spans the required vertices, has no cycle, and has the expected minimum total weight.

## 16. Practice progression

### Beginner

1. Implement Dijkstra and reject negative edge weights.
2. Add predecessor tracking and reconstruct one shortest path.
3. Implement Bellman–Ford and test a reachable negative cycle.
4. Implement Floyd–Warshall for a small adjacency matrix.

### Intermediate

5. Implement DSU with path compression and union by size.
6. Implement Kruskal and detect disconnected input.
7. Implement Prim using a min-heap.
8. Compare Prim and Kruskal on the same connected weighted graphs.

### Challenge

9. Implement Kosaraju's SCC algorithm.
10. Implement Tarjan's SCC algorithm and compare the component sets.
11. Implement bridges and articulation points using DFS low-link values.
12. Implement shortest paths in a DAG using topological order.
13. Build a minimum spanning forest for a disconnected graph.

## 17. Mini project — Network Route and Resilience Analyzer

<badge color="info">Repository demonstration</badge>

Build a command-line tool that analyzes a small, fictional network graph.

### Suggested features

- Read vertices and weighted edges from a structured input file.
- Validate vertex references and edge weights.
- Calculate shortest paths from a selected source.
- Select the shortest-path algorithm based on declared graph constraints.
- Calculate an MST or minimum spanning forest for undirected weighted input.
- Report bridges or articulation points for undirected topology.
- Display disconnected or unreachable components clearly.

### Suggested test cases

- A connected network.
- A network with an isolated node.
- A directed dependency graph.
- A graph with a negative edge but no negative cycle.
- A graph with a reachable negative cycle.
- An undirected graph with redundant routes.
- A graph whose MST has tied edge weights.

### Engineering boundaries

This is a repository-designed learning exercise. Its graph is a simplified model; it does not discover actual infrastructure topology, measure network latency, or validate a production routing configuration.

## 18. Interview questions

1. When should you use BFS instead of Dijkstra?
2. Why does Dijkstra require non-negative edge weights?
3. What is edge relaxation?
4. How does Bellman–Ford detect a reachable negative cycle?
5. Why does Bellman–Ford perform up to V - 1 relaxation passes?
6. What problem does Floyd–Warshall solve?
7. What is the difference between an MST and a shortest-path tree?
8. How do Prim and Kruskal differ?
9. What does DSU's find operation return?
10. Why are path compression and union by size useful?
11. What is a strongly connected component?
12. What is the purpose of the transposed graph in Kosaraju's algorithm?
13. What are bridges and articulation points?
14. What is a low-link value used for?
15. Why can a DAG shortest-path algorithm support negative edge weights?
16. How would you validate an MST returned by an implementation?

## 19. Completion checklist

- [ ] I can choose a shortest-path algorithm based on graph assumptions.
- [ ] I can implement Dijkstra and explain its non-negative-weight requirement.
- [ ] I can reconstruct a shortest path using predecessor information.
- [ ] I can implement Bellman–Ford and explain its negative-cycle check.
- [ ] I understand the all-pairs role of Floyd–Warshall.
- [ ] I can distinguish an MST from a shortest-path tree.
- [ ] I can implement DSU with path compression and union by size.
- [ ] I can explain Kruskal and Prim.
- [ ] I understand strongly connected components.
- [ ] I can explain bridges, articulation points, and low-link values.
- [ ] I understand DAG shortest paths.
- [ ] I can test disconnected graphs and edge cases.
- [ ] I can analyze time and space in terms of V and E.

## 20. Sources and quality notes

### Python implementation references

- [Python heapq — Heap queue algorithm](https://docs.python.org/3/library/heapq.html)
- [Python collections — deque](https://docs.python.org/3/library/collections.html#collections.deque)

These official references document Python data structures used in graph implementations. They do not serve as proofs of the graph algorithms' correctness.

### Content boundaries

- **Algorithmic concepts:** The chapter presents standard graph algorithms and states important graph assumptions and limitations.
- **Python implementation:** The Dijkstra, Bellman–Ford, Floyd–Warshall, and Kruskal examples are instructional implementations, not production routing or network-management software.
- **Repository design decision:** The Network Route and Resilience Analyzer is a learning project designed for this repository.
- **Not implied:** A graph model built for an exercise is not a live representation of cloud, Kubernetes, or physical network infrastructure. Real integrations require authoritative inventory and topology data, validation, and operational safeguards.

---

[← Previous: 14 — Dynamic Programming](../14%20Dynamic%20Programming/README.md) · [Complete DSA home](../README.md) · [Next: 16 — Advanced Data Structures](../16%20Advanced%20Data%20Structures/README.md)
