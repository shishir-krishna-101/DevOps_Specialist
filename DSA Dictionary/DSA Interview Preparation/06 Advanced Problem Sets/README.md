# 06 — Advanced Problem Sets

Advanced interview problems ask you to reason about more than one operation or one pass through an input. They often require a data structure that preserves information across updates, a graph algorithm with explicit weight assumptions, or a dynamic-programming state that captures a larger decision space.

> **Learning goal:** Design, implement, and justify advanced solutions while being precise about invariants, operation costs, input assumptions, and edge cases.

This is a curated set of advanced exercises, not an exhaustive interview syllabus. A correct implementation is only part of the task: be ready to explain why the chosen structure fits the operations and what trade-offs it introduces.

---

## How to approach advanced problems

Before coding, answer these questions:

1. **What operations must be supported?** One query, repeated queries, updates, insertions, deletions, or a stream?
2. **What are the constraints?** Input size, value range, graph weights, duplicate behavior, and memory limits?
3. **What information must be retained?** A frontier, a tree of aggregates, connectivity components, cached entries, or subproblem results?
4. **What is the invariant?** State what remains true after every update or algorithm step.
5. **What are the operation costs?** Include construction, updates, queries, and output generation separately.
6. **What assumptions are essential?** For example, Dijkstra requires non-negative edge weights.
7. **How will you test it?** Include empty inputs, repeated operations, disconnected components, boundary indices, and adversarial ordering.

## Problem map

| # | Problem | Main technique | Core challenge |
|---:|---|---|---|
| 1 | Implement a Trie | Prefix tree | Share prefixes and distinguish complete words |
| 2 | Dynamic connectivity | Disjoint Set Union | Merge components efficiently |
| 3 | Single-source shortest paths | Dijkstra + min-heap | Finalize shortest distances under weight constraints |
| 4 | Mutable range sums | Fenwick tree | Support updates and prefix/range queries |
| 5 | Mutable range minimum | Segment tree | Maintain an associative range aggregate |
| 6 | LRU cache | Hash map + ordered mapping | Maintain recency in constant expected time |
| 7 | Median from a data stream | Two heaps | Maintain balanced lower and upper halves |
| 8 | N-Queens | Backtracking + pruning | Search a constrained state space |
| 9 | Edit distance | Dynamic programming | Model insert, delete, and replace transitions |
| 10 | Weighted interval scheduling | DP + binary search | Combine compatible jobs optimally |
| 11 | Maximum profit with prerequisites? | — | Reserved for future expansion |

This set contains ten complete problems. The final row is intentionally not a placeholder exercise; additional advanced problems can be added in a later revision after their contracts and solutions are designed.

---

## 1. Implement a Trie

**Pattern:** Prefix tree

### Problem

Implement a Trie that supports:

- \`insert(word)\`: add a word.
- \`search(word)\`: return whether the complete word has been inserted.
- \`starts_with(prefix)\`: return whether at least one inserted word begins with the prefix.

**Contract and assumptions**

- Words are strings; any Python string characters are accepted.
- Matching is exact and case-sensitive.
- Inserting the same word more than once has no additional effect.
- The empty string is a valid word and a valid prefix.
- No deletion operation is required.

**Example**

~~~python
trie = Trie()
trie.insert("platform")
trie.insert("platforms")

assert trie.search("platform") is True
assert trie.search("platforms") is True
assert trie.search("plat") is False
assert trie.starts_with("plat") is True
assert trie.starts_with("cloud") is False
~~~

### Hint

Each node maps a character to a child node and records whether a complete word ends at that node. Reaching a node is not enough for \`search\`; the terminal marker must also be set.

### Worked solution

~~~python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_word = False


class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root

        for character in word:
            if character not in node.children:
                node.children[character] = TrieNode()
            node = node.children[character]

        node.is_word = True

    def _find_node(self, text):
        node = self.root

        for character in text:
            if character not in node.children:
                return None
            node = node.children[character]

        return node

    def search(self, word):
        node = self._find_node(word)
        return node is not None and node.is_word

    def starts_with(self, prefix):
        return self._find_node(prefix) is not None


trie = Trie()
trie.insert("platform")
trie.insert("platforms")
trie.insert("")

assert trie.search("platform") is True
assert trie.search("platforms") is True
assert trie.search("plat") is False
assert trie.starts_with("plat") is True
assert trie.starts_with("cloud") is False
assert trie.search("") is True
assert trie.starts_with("") is True
~~~

### Why it works

A path from the root represents a prefix. Insertion creates any missing nodes and marks the final node as a complete word. Searching follows the same path but additionally checks the terminal marker. Prefix lookup only needs to establish that the path exists.

Let L be the length of the supplied word or prefix:

- **Insert:** O(L) expected, assuming average constant-time dictionary operations.
- **Search:** O(L) expected.
- **Prefix lookup:** O(L) expected.
- **Space:** O(total number of characters across inserted words) in the worst case; shared prefixes reduce the number of nodes.

**Common mistake:** Returning true from \`search\` whenever a path exists. That incorrectly treats a prefix as a complete word.

**When to use it:** Prefix lookup, autocomplete foundations, routing-table-like prefix structures, and dictionaries with many shared prefixes. For a small collection with few prefix queries, a Trie may use more memory than a simpler set.

---

## 2. Dynamic connectivity with Disjoint Set Union

**Pattern:** Disjoint Set Union (Union-Find)

### Problem

There are \`n\` vertices labeled from 0 to \`n - 1\`. Process a sequence of undirected edge additions. After each addition, determine whether the two endpoints are already connected after that edge has been added.

Implement a structure with:

- \`find(vertex)\`: return the representative of its component.
- \`union(first, second)\`: merge two components and return whether a merge occurred.
- \`connected(first, second)\`: test whether two vertices share a component.

**Contract and assumptions**

- \`n\` is a non-negative integer.
- Vertices must be in the range \`0 <= vertex < n\`.
- The structure supports edge additions, not arbitrary edge removals.
- Repeated unions are allowed.

**Example**

~~~python
dsu = DisjointSetUnion(5)

assert dsu.union(0, 1) is True
assert dsu.union(1, 2) is True
assert dsu.connected(0, 2) is True
assert dsu.union(0, 2) is False
assert dsu.connected(0, 4) is False
~~~

### Hint

Store a parent for each vertex and a size for each root. Use path compression in \`find\`, and attach the smaller component below the larger component during \`union\`.

### Worked solution

~~~python
class DisjointSetUnion:
    def __init__(self, n):
        if n < 0:
            raise ValueError("n cannot be negative")

        self.parent = list(range(n))
        self.size = [1] * n
        self.components = n

    def _validate(self, vertex):
        if not 0 <= vertex < len(self.parent):
            raise IndexError("vertex is out of range")

    def find(self, vertex):
        self._validate(vertex)

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
        self.components -= 1
        return True

    def connected(self, first, second):
        return self.find(first) == self.find(second)


dsu = DisjointSetUnion(5)
assert dsu.components == 5
assert dsu.union(0, 1) is True
assert dsu.union(1, 2) is True
assert dsu.connected(0, 2) is True
assert dsu.union(0, 2) is False
assert dsu.connected(0, 4) is False
assert dsu.components == 3
assert DisjointSetUnion(0).components == 0
~~~

### Why it works

Every root represents one connected component. \`find\` follows parent links to the root and compresses the traversed path, making future finds faster. \`union\` connects two different roots; union by size ensures the smaller tree is attached below the larger one. If both vertices already have the same root, their components are already joined and no update is required.

With path compression and union by size, a sequence of operations has amortized O(α(n)) time per operation, where α is the inverse Ackermann function and grows extremely slowly.

- **Find / union / connected:** O(α(n)) amortized per operation.
- **Space:** O(n).

**Important limitation:** This structure supports incremental connectivity. It does not directly support deleting an arbitrary edge, because a deleted edge may split a component and parent links do not retain enough information to reconstruct that split.

**DevOps/SRE application:** Track connected components while adding network links, service relationships, or dependency edges in an offline analysis.

---

## 3. Single-source shortest paths with Dijkstra

**Pattern:** Weighted graph + min-heap

### Problem

Given a directed, weighted graph with vertices labeled from 0 to \`n - 1\`, return the shortest distance from a source vertex to every vertex.

**Contract and assumptions**

- The graph is represented as an adjacency list: \`graph[u]\` contains pairs \`(v, weight)\`.
- \`n\` is non-negative; the source must be a valid vertex when \`n > 0\`.
- Every edge endpoint is valid.
- Edge weights are non-negative integers.
- Unreachable vertices have distance \`None\`.
- The graph may contain parallel edges and cycles.

**Example**

~~~python
graph = [
    [(1, 4), (2, 1)],
    [(3, 1)],
    [(1, 2), (3, 5)],
    [],
]
# From source 0: [0, 3, 1, 4]
~~~

### Hint

Initialize the source distance to zero and push it into a min-heap. When a vertex is popped, ignore stale heap entries whose distance is no longer current. Relax each outgoing edge.

### Worked solution

~~~python
import heapq


def dijkstra(graph, source):
    n = len(graph)

    if not 0 <= source < n:
        raise ValueError("source is out of range")

    for neighbors in graph:
        for neighbor, weight in neighbors:
            if not 0 <= neighbor < n:
                raise ValueError("edge endpoint is out of range")
            if weight < 0:
                raise ValueError("Dijkstra requires non-negative edge weights")

    distances = [None] * n
    distances[source] = 0
    pending = [(0, source)]

    while pending:
        current_distance, vertex = heapq.heappop(pending)

        if current_distance != distances[vertex]:
            continue

        for neighbor, weight in graph[vertex]:
            candidate = current_distance + weight
            known = distances[neighbor]

            if known is None or candidate < known:
                distances[neighbor] = candidate
                heapq.heappush(pending, (candidate, neighbor))

    return distances


graph = [
    [(1, 4), (2, 1)],
    [(3, 1)],
    [(1, 2), (3, 5)],
    [],
]
assert dijkstra(graph, 0) == [0, 3, 1, 4]
assert dijkstra([[], []], 0) == [0, None]
assert dijkstra([[(1, 0)], []], 0) == [0, 0]
assert dijkstra([[(1, 8), (1, 2)], []], 0) == [0, 2]
~~~

### Why it works

Dijkstra repeatedly selects the unsettled vertex with the smallest tentative distance. With non-negative edge weights, once that minimum-distance entry is removed from the min-heap and is current, no later path through an unsettled vertex can produce a shorter distance to it. Relaxation then improves neighboring tentative distances whenever a shorter route is found.

The stale-entry check is necessary because the implementation pushes a new heap entry when a better distance is found instead of searching inside the heap to update an existing entry.

Let V be the number of vertices and E the number of directed edges:

- **Time:** O((V + E) log V) for a binary heap with lazy duplicate entries; a looser common bound is O((V + E) log E) depending on graph representation and edge count.
- **Auxiliary space:** O(V + E) including the heap in the worst case; the adjacency list is input storage.

**Critical warning:** Dijkstra is not correct for graphs with negative edge weights. Use an algorithm designed for the relevant weight model, such as Bellman–Ford when negative edges are permitted and negative-cycle behavior must be considered.

**DevOps/SRE application:** Model a network or service graph with non-negative link costs and calculate least-cost routes. The cost model must be explicitly defined; this is an algorithmic demonstration, not a claim about a production routing protocol.

---

## 4. Mutable range sums with a Fenwick tree

**Pattern:** Binary Indexed Tree

### Problem

Maintain a list of integers while supporting:

- Add a delta to one index.
- Return the sum of a prefix.
- Return the sum of a half-open range \`[left, right)\`.

**Contract and assumptions**

- The initial values are integers.
- Indices are zero-based.
- \`prefix_sum(end)\` returns the sum over indices from 0 through \`end - 1\`.
- \`range_sum(left, right)\` uses a half-open range and requires \`0 <= left <= right <= n\`.
- Updates use addition, not assignment.

**Example**

~~~python
tree = FenwickTree([2, 4, 1, 7])

assert tree.prefix_sum(3) == 7
assert tree.range_sum(1, 4) == 12

tree.add(2, 5)  # index 2 changes from 1 to 6
assert tree.range_sum(1, 4) == 17
~~~

### Hint

Internally use one-based indices. Each Fenwick node stores the sum of a range determined by the least significant set bit. Move upward by adding that bit during updates and downward by subtracting it during prefix queries.

### Worked solution

~~~python
class FenwickTree:
    def __init__(self, values):
        self.n = len(values)
        self.tree = [0] * (self.n + 1)

        for index, value in enumerate(values):
            self.add(index, value)

    def add(self, index, delta):
        if not 0 <= index < self.n:
            raise IndexError("index is out of range")

        position = index + 1
        while position <= self.n:
            self.tree[position] += delta
            position += position & -position

    def prefix_sum(self, end):
        if not 0 <= end <= self.n:
            raise IndexError("prefix boundary is out of range")

        total = 0
        position = end

        while position > 0:
            total += self.tree[position]
            position -= position & -position

        return total

    def range_sum(self, left, right):
        if not 0 <= left <= right <= self.n:
            raise IndexError("invalid range")

        return self.prefix_sum(right) - self.prefix_sum(left)


tree = FenwickTree([2, 4, 1, 7])
assert tree.prefix_sum(0) == 0
assert tree.prefix_sum(3) == 7
assert tree.range_sum(1, 4) == 12
tree.add(2, 5)
assert tree.range_sum(1, 4) == 17
assert tree.range_sum(2, 2) == 0
assert FenwickTree([]).prefix_sum(0) == 0
~~~

### Why it works

The least significant set bit identifies the size of the block represented by a Fenwick node. A point update adds its delta to every Fenwick block that contains the index. A prefix query decomposes the prefix into disjoint Fenwick blocks and adds their stored sums. Subtracting two prefixes gives the requested half-open range.

- **Construction in this implementation:** O(n log n).
- **Point update:** O(log n).
- **Prefix query:** O(log n).
- **Range sum:** O(log n).
- **Auxiliary space:** O(n).

**Common mistake:** Mixing inclusive and half-open boundaries. This implementation defines \`prefix_sum(end)\` over \`[0, end)\` and \`range_sum(left, right)\` over \`[left, right)\`.

**When to use it:** Frequent point additions and prefix/range-sum queries. If updates assign a new value rather than add a delta, store the current values and compute the difference before calling \`add\`.

---

## 5. Mutable range minimum with a segment tree

**Pattern:** Segment tree

### Problem

Maintain an integer array with:

- Point assignment: replace the value at an index.
- Range minimum: return the minimum over a non-empty half-open range \`[left, right)\`.

**Contract and assumptions**

- Values are integers.
- Indices are zero-based.
- The query range must be non-empty and satisfy \`0 <= left < right <= n\`.
- The input list is copied; updates do not modify the caller's list.

**Example**

~~~python
tree = SegmentTree([5, 2, 7, 3, 6])

assert tree.range_min(1, 4) == 2
tree.update(1, 8)
assert tree.range_min(1, 4) == 3
~~~

### Hint

Use an iterative tree stored in an array of length \`2 * n\`. Leaves occupy the second half. Each internal node stores the minimum of its two children.

### Worked solution

~~~python
class SegmentTree:
    def __init__(self, values):
        self.n = len(values)
        self.tree = [float("inf")] * (2 * self.n)

        for index, value in enumerate(values):
            self.tree[self.n + index] = value

        for index in range(self.n - 1, 0, -1):
            self.tree[index] = min(
                self.tree[2 * index],
                self.tree[2 * index + 1]
            )

    def update(self, index, value):
        if not 0 <= index < self.n:
            raise IndexError("index is out of range")

        position = self.n + index
        self.tree[position] = value
        position //= 2

        while position > 0:
            self.tree[position] = min(
                self.tree[2 * position],
                self.tree[2 * position + 1]
            )
            position //= 2

    def range_min(self, left, right):
        if not 0 <= left < right <= self.n:
            raise IndexError("query must be a non-empty valid range")

        left += self.n
        right += self.n
        answer = float("inf")

        while left < right:
            if left % 2 == 1:
                answer = min(answer, self.tree[left])
                left += 1

            if right % 2 == 1:
                right -= 1
                answer = min(answer, self.tree[right])

            left //= 2
            right //= 2

        return answer


tree = SegmentTree([5, 2, 7, 3, 6])
assert tree.range_min(1, 4) == 2
assert tree.range_min(0, 5) == 2
tree.update(1, 8)
assert tree.range_min(1, 4) == 3
assert tree.range_min(0, 1) == 5
assert SegmentTree([-4]).range_min(0, 1) == -4
~~~

### Why it works

Every leaf stores one input value. Every internal node stores the minimum of its children, so it represents the minimum of the corresponding segment. A point update changes one leaf and recomputes every ancestor. A range query moves upward while consuming complete segments that lie inside the requested half-open interval. These consumed segments are disjoint and cover the query range.

- **Build:** O(n).
- **Point assignment:** O(log n).
- **Range minimum:** O(log n).
- **Auxiliary space:** O(n).

The iterative representation works with arbitrary n; it does not require padding to a power of two for this query/update implementation.

**Common mistake:** Updating a leaf but not recomputing its ancestors. Future queries would then read stale aggregate values.

**Generalization:** Segment trees can maintain other associative aggregates, such as sum, minimum with index, maximum, or greatest common divisor. The combine operation and identity must match the query.

---

## 6. LRU cache

**Pattern:** Hash map + ordered mapping

### Problem

Implement a Least Recently Used (LRU) cache with a fixed non-negative capacity.

Support:

- \`get(key)\`: return the value and mark the key as most recently used; return -1 if absent.
- \`put(key, value)\`: insert or update a key and mark it most recently used. If capacity is exceeded, evict the least recently used key.

**Contract and assumptions**

- Keys are hashable.
- Capacity is a non-negative integer.
- Values may be any Python object.
- The cache uses -1 as the missing-key return value; this API therefore does not distinguish a stored value of -1 from a missing key by return value alone.
- The implementation is not designed as a thread-safe cache.

**Example**

~~~python
cache = LRUCache(2)
cache.put("a", 1)
cache.put("b", 2)

assert cache.get("a") == 1
cache.put("c", 3)  # evicts "b"

assert cache.get("b") == -1
assert cache.get("c") == 3
~~~

### Hint

Python's \`OrderedDict\` supports moving a key to either end. Keep the least recently used item at the beginning and the most recently used item at the end.

### Worked solution

~~~python
from collections import OrderedDict


class LRUCache:
    def __init__(self, capacity):
        if capacity < 0:
            raise ValueError("capacity cannot be negative")

        self.capacity = capacity
        self.items = OrderedDict()

    def get(self, key):
        if key not in self.items:
            return -1

        self.items.move_to_end(key)
        return self.items[key]

    def put(self, key, value):
        if self.capacity == 0:
            return

        if key in self.items:
            self.items[key] = value
            self.items.move_to_end(key)
        else:
            self.items[key] = value

        if len(self.items) > self.capacity:
            self.items.popitem(last=False)


cache = LRUCache(2)
cache.put("a", 1)
cache.put("b", 2)
assert cache.get("a") == 1
cache.put("c", 3)
assert cache.get("b") == -1
assert cache.get("c") == 3
cache.put("a", 10)
assert cache.get("a") == 10

zero_capacity = LRUCache(0)
zero_capacity.put("x", 1)
assert zero_capacity.get("x") == -1
~~~

### Why it works

The ordered mapping maintains a recency order. A successful read and every write move the key to the most-recent end. When the number of entries exceeds capacity, removing the first item evicts the least recently used key.

- **Get:** O(1) expected.
- **Put:** O(1) expected.
- **Space:** O(capacity).

The expected constant-time statement relies on hash-table operations and the documented ordered-mapping operations.

**Common mistake:** Updating a value without updating its recency. An updated key is considered recently used under this contract.

**Production note:** This is an interview implementation, not a complete production cache. Real caches may also need expiration, concurrency control, memory limits, metrics, and explicit handling of a missing value that cannot collide with a valid stored value.

---

## 7. Median from a data stream

**Pattern:** Two heaps

### Problem

Design a structure that accepts integers one at a time and returns the median of all values received so far.

Support:

- \`add(value)\`: insert one integer.
- \`median()\`: return the current median.

**Contract and assumptions**

- Values are integers.
- Calling \`median\` before inserting a value raises \`ValueError\`.
- For an even number of values, return the arithmetic mean of the two middle values as a float.

**Example**

~~~python
stream = MedianFinder()
stream.add(5)
stream.add(1)
assert stream.median() == 3.0
stream.add(9)
assert stream.median() == 5.0
~~~

### Hint

Keep the lower half in a max-heap and the upper half in a min-heap. Python's \`heapq\` is a min-heap, so store negated values for the lower half. Keep the heap sizes equal or let the lower half have one extra value.

### Worked solution

~~~python
import heapq


class MedianFinder:
    def __init__(self):
        self.lower = []  # Negated values: max-heap behavior.
        self.upper = []  # Ordinary min-heap.

    def add(self, value):
        if not self.lower or value <= -self.lower[0]:
            heapq.heappush(self.lower, -value)
        else:
            heapq.heappush(self.upper, value)

        if len(self.lower) > len(self.upper) + 1:
            moved = -heapq.heappop(self.lower)
            heapq.heappush(self.upper, moved)
        elif len(self.upper) > len(self.lower):
            moved = heapq.heappop(self.upper)
            heapq.heappush(self.lower, -moved)

    def median(self):
        if not self.lower:
            raise ValueError("no values have been added")

        if len(self.lower) > len(self.upper):
            return float(-self.lower[0])

        return (-self.lower[0] + self.upper[0]) / 2.0


stream = MedianFinder()
try:
    stream.median()
    raise AssertionError("median() should reject an empty stream")
except ValueError:
    pass

stream.add(5)
stream.add(1)
assert stream.median() == 3.0
stream.add(9)
assert stream.median() == 5.0
stream.add(2)
assert stream.median() == 3.5
stream.add(-10)
assert stream.median() == 2.0
~~~

### Why it works

The lower heap stores the smaller half and the upper heap stores the larger half. The ordering invariant is that every lower-half value is no greater than every upper-half value. The size invariant is that the lower heap has either the same number of elements as the upper heap or exactly one more.

Therefore, the maximum of the lower half is the median for odd-sized streams. For even-sized streams, the median is the average of the lower maximum and upper minimum.

- **Add:** O(log n).
- **Median:** O(1).
- **Auxiliary space:** O(n).

**Common mistake:** Balancing heap sizes without preserving the ordering invariant. Both invariants are required.

---

## 8. N-Queens

**Pattern:** Backtracking + constraint pruning

### Problem

Place n queens on an n × n chessboard so that no two queens attack one another. Return every distinct solution as a list of board rows, where Q marks a queen and . marks an empty square.

**Contract and assumptions**

- n is a non-negative integer.
- Return all solutions.
- For n = 0, return one empty board representation: \`[[]]\`.
- Each solution has exactly one queen in every row and every column.

**Example**

~~~python
solutions = solve_n_queens(4)
assert len(solutions) == 2
~~~

### Hint

Place one queen per row. Track occupied columns and the two diagonal identifiers. A candidate square is safe only if none of its identifiers is already occupied.

### Worked solution

~~~python
def solve_n_queens(n):
    if n < 0:
        raise ValueError("n cannot be negative")
    if n == 0:
        return [[]]

    solutions = []
    queen_columns = [-1] * n
    used_columns = set()
    used_descending = set()
    used_ascending = set()

    def backtrack(row):
        if row == n:
            board = []

            for queen_column in queen_columns:
                board.append(
                    "." * queen_column
                    + "Q"
                    + "." * (n - queen_column - 1)
                )

            solutions.append(board)
            return

        for column in range(n):
            descending = row - column
            ascending = row + column

            if (
                column in used_columns
                or descending in used_descending
                or ascending in used_ascending
            ):
                continue

            queen_columns[row] = column
            used_columns.add(column)
            used_descending.add(descending)
            used_ascending.add(ascending)

            backtrack(row + 1)

            used_columns.remove(column)
            used_descending.remove(descending)
            used_ascending.remove(ascending)
            queen_columns[row] = -1

    backtrack(0)
    return solutions


assert solve_n_queens(0) == [[]]
assert len(solve_n_queens(1)) == 1
assert solve_n_queens(2) == []
assert solve_n_queens(3) == []
assert len(solve_n_queens(4)) == 2
~~~

### Why it works

The search places exactly one queen per row. The column set prevents two queens from sharing a column. For a cell at (row, column), the values row - column and row + column identify its two diagonal directions; equal identifiers mean two queens share a diagonal.

A branch is explored only when the new queen does not conflict with any previously placed queen. Every valid board has exactly one queen in each row, so the row-by-row search considers its placement sequence. Safe pruning removes only partial boards that cannot be extended to a valid solution.

- **Time:** Exponential; a simple upper-bound discussion is O(n!) candidate column permutations, with pruning reducing the explored search in practice. Constructing all returned boards adds output cost.
- **Auxiliary space:** O(n) for recursion and constraint sets, excluding the returned solutions.

**Common mistake:** Forgetting to remove a column or diagonal from the sets during backtracking. That contaminates sibling branches.

**Interview discussion:** Ask whether the task requires one solution, the number of solutions, or every board. Returning every board necessarily incurs output memory proportional to the output size.

---

## 9. Edit distance

**Pattern:** Dynamic programming

### Problem

Given two strings, return the minimum number of single-character operations needed to transform the first string into the second.

Allowed operations:

- Insert one character.
- Delete one character.
- Replace one character.

**Contract and assumptions**

- Inputs are Python strings.
- Comparison is exact and case-sensitive.
- Return an integer distance.
- No transposition operation is included.

**Example**

~~~python
assert edit_distance("kitten", "sitting") == 3
~~~

### Hint

Let dp[i][j] be the minimum edits required to transform the first i characters of one string into the first j characters of the other. Consider insertion, deletion, and replacement/match.

### Worked solution

~~~python
def edit_distance(first, second):
    # Keep the shorter string as the DP column dimension.
    if len(second) > len(first):
        first, second = second, first

    previous = list(range(len(second) + 1))

    for row, first_character in enumerate(first, start=1):
        current = [row] + [0] * len(second)

        for column, second_character in enumerate(second, start=1):
            insertion = current[column - 1] + 1
            deletion = previous[column] + 1
            substitution = previous[column - 1] + (
                first_character != second_character
            )

            current[column] = min(
                insertion,
                deletion,
                substitution
            )

        previous = current

    return previous[-1]


assert edit_distance("", "") == 0
assert edit_distance("kitten", "sitting") == 3
assert edit_distance("flaw", "lawn") == 2
assert edit_distance("same", "same") == 0
assert edit_distance("abc", "") == 3
assert edit_distance("", "abc") == 3
~~~

### Why it works

The empty-prefix base cases require j insertions to build a target prefix of length j and i deletions to reduce a source prefix of length i to empty. For non-empty prefixes, the final operation is one of insertion, deletion, or substitution/match. The recurrence chooses the least expensive valid predecessor state.

Only the previous row and current row are needed to compute the next row, so the full two-dimensional table is not retained.

- **Time:** O(m × n), where m and n are the string lengths.
- **Auxiliary space:** O(min(m, n)) because the shorter string is used as the column dimension.

**Common mistake:** Using the wrong previous-row or current-row cell for an operation. Draw a small DP table and label the meaning of each state before writing the recurrence.

---

## 10. Weighted interval scheduling

**Pattern:** Dynamic programming + binary search

### Problem

Given jobs represented as \`(start, end, profit)\`, choose a set of non-overlapping jobs that maximizes total profit.

**Contract and assumptions**

- Each job has integer start, end, and profit values.
- Every job has positive duration: start < end.
- Jobs are compatible when the earlier job's end is less than or equal to the later job's start.
- Profit may be negative; choosing no jobs is allowed and yields zero.
- Return the maximum achievable profit, not the selected job list.

**Example**

~~~python
jobs = [
    (1, 3, 50),
    (3, 5, 20),
    (6, 9, 100),
    (2, 8, 200),
]
assert maximum_job_profit(jobs) == 200
~~~

### Hint

Sort jobs by finishing time. For each job, find the last earlier job whose end is no greater than this job's start. Compare skipping the job with taking its profit plus the best compatible prefix solution.

### Worked solution

~~~python
from bisect import bisect_right


def maximum_job_profit(jobs):
    for start, end, profit in jobs:
        if start >= end:
            raise ValueError("each job must have positive duration")

    ordered = sorted(jobs, key=lambda job: job[1])
    ends = [job[1] for job in ordered]

    # dp[i] is the best profit using the first i jobs in sorted order.
    dp = [0] * (len(ordered) + 1)

    for index, (start, end, profit) in enumerate(ordered):
        # Search only jobs before this one.
        compatible_count = bisect_right(
            ends, start, 0, index
        )

        take = profit + dp[compatible_count]
        skip = dp[index]
        dp[index + 1] = max(skip, take)

    return dp[-1]


assert maximum_job_profit([
    (1, 3, 50),
    (3, 5, 20),
    (6, 9, 100),
    (2, 8, 200),
]) == 200
assert maximum_job_profit([]) == 0
assert maximum_job_profit([(1, 2, 5), (2, 3, 6)]) == 11
assert maximum_job_profit([(1, 5, 10), (2, 3, 100)]) == 100
assert maximum_job_profit([(1, 2, -5), (2, 3, -2)]) == 0
~~~

### Why it works

After sorting by end time, every job before the current job has an end time no greater than or equal to the current job's end. Binary search finds the number of earlier jobs whose end is at most the current start. The DP value at that count is the best profit among all jobs compatible with the current job.

An optimal solution either skips the current job, retaining the previous best, or takes it, adding its profit to the best compatible earlier solution. The recurrence considers both possibilities.

- **Time:** O(n log n) for sorting and one binary search per job.
- **Auxiliary space:** O(n) for the sorted jobs, end-time list, and DP array.

**Common mistake:** Searching the full end-time list, including the current job or later jobs. The search upper bound must be the current index.

**Boundary rule:** This problem treats jobs with end == next_start as compatible. If a different scheduling contract requires a gap, change the compatibility condition and binary-search boundary.

---

## Complexity and structure comparison

| Structure / algorithm | Build or setup | Main operation | Typical auxiliary space |
|---|---:|---:|---:|
| Trie | O(total inserted characters) | O(L) per word/prefix operation | O(total characters) |
| Disjoint Set Union | O(n) | O(α(n)) amortized | O(n) |
| Dijkstra with heap | O(V) initialization | O((V + E) log V) overall | O(V + E) |
| Fenwick tree | O(n log n) here | O(log n) update/query | O(n) |
| Segment tree | O(n) | O(log n) update/query | O(n) |
| LRU cache | O(1) empty setup | O(1) expected get/put | O(capacity) |
| Median from stream | O(1) empty setup | O(log n) insert, O(1) median | O(n) |
| N-Queens | — | Exponential search | O(n), excluding output |
| Edit distance | — | O(mn) | O(min(m, n)) |
| Weighted interval scheduling | O(n log n) | O(log n) predecessor search per job | O(n) |

These are asymptotic analyses for the implementations and contracts in this file. Constants, Python object overhead, input representation, and output size still matter in practical workloads.

## Testing checklist

For each advanced implementation, test more than the happy path:

- **Trie:** Empty word, prefix that is not a word, repeated insertion, absent prefix.
- **DSU:** Repeated union, isolated vertices, zero vertices, invalid identifiers.
- **Dijkstra:** Disconnected graph, zero-weight edge, parallel edges, negative-edge rejection.
- **Fenwick tree:** Empty input, first and last index, empty range, repeated updates.
- **Segment tree:** One element, full range, one-element range, update to a new minimum.
- **LRU cache:** Capacity zero, capacity one, read changes recency, update changes recency.
- **Median stream:** Empty stream, odd and even counts, negative values, repeated values.
- **N-Queens:** n = 0, n = 1, impossible small sizes, known n = 4 result count.
- **Edit distance:** Empty strings, identical strings, insertion-only and deletion-only cases.
- **Weighted scheduling:** Empty input, touching endpoints, overlapping jobs, negative profits.

For a data structure, test sequences of operations, not just isolated calls. A bug may only appear after several updates have changed its internal state.

## Suggested practice schedule

| Session | Work |
|---|---|
| 1 | Trie and Disjoint Set Union |
| 2 | Dijkstra and graph-weight assumptions |
| 3 | Fenwick tree and segment tree |
| 4 | LRU cache and median from a stream |
| 5 | N-Queens and pruning correctness |
| 6 | Edit distance and weighted interval scheduling |
| 7 | Re-implement two structures from memory and explain their invariants |

For each problem, first implement the core version shown here. Then change one requirement—such as supporting deletion, returning an actual path, or reconstructing the selected jobs—and explain what additional state is needed.

## Completion checklist

- [ ] I can state the operations and constraints before selecting a structure.
- [ ] I can explain the invariant maintained by each data structure.
- [ ] I can distinguish expected, amortized, and worst-case complexity.
- [ ] I can explain why Dijkstra's non-negative weight condition matters.
- [ ] I can implement a Fenwick tree and segment tree without mixing range conventions.
- [ ] I can explain both ordering and size invariants in a two-heap median structure.
- [ ] I can justify every backtracking pruning rule.
- [ ] I can derive the DP state, base cases, and recurrence.
- [ ] I can test sequences of operations and adversarial inputs.
- [ ] I can discuss limitations and alternative approaches.

## Navigation

- [DSA Interview Preparation — Main Guide](../README.md)
- [01 — Interview Roadmap](../01%20Interview%20Roadmap/README.md)
- [03 — Pattern Recognition](../03%20Pattern%20Recognition/README.md)
- [04 — Easy Problem Sets](../04%20Easy%20Problem%20Sets/README.md)
- [05 — Medium Problem Sets](../05%20Medium%20Problem%20Sets/README.md)
- [Complete DSA — Advanced Graph Algorithms](../../Complete%20DSA/15%20Advanced%20Graph%20Algorithms/README.md)
- [Complete DSA — Advanced Data Structures](../../Complete%20DSA/16%20Advanced%20Data%20Structures/README.md)
- [Complete DSA — Interview Patterns](../../Complete%20DSA/17%20Interview%20Patterns/README.md)
- [Complete DSA — Practice Problems](../../Complete%20DSA/18%20DSA%20Practice%20Problems/README.md)

## Official Python references

These official references support the standard-library APIs used in the examples:

- [collections — Container datatypes](https://docs.python.org/3/library/collections.html)
- [heapq — Heap queue algorithm](https://docs.python.org/3/library/heapq.html)
- [bisect — Array bisection algorithm](https://docs.python.org/3/library/bisect.html)
- [Built-in Types — Python documentation](https://docs.python.org/3/library/stdtypes.html)

The algorithms and problem contracts in this file are educational implementations and repository design decisions. The Python documentation verifies the referenced language and standard-library APIs; it does not prescribe these interview problems or guarantee interview outcomes.
