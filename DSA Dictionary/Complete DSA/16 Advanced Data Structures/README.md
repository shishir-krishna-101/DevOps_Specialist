# 16 — Advanced Data Structures

Advanced data structures help when the basic choices—arrays, lists, hash tables, stacks, queues, trees, and heaps—do not provide the operations a problem needs efficiently.

The important skill is not memorizing every structure. It is learning to identify the operations a workload performs, then choosing a structure whose update, lookup, and query costs fit that workload.

> **Learning goal:** Understand the purpose, invariants, operations, trade-offs, and common use cases of several advanced data structures. Implement the core structures yourself before relying on a library.

---

## 1. How to approach an advanced data structure

Before choosing a structure, write down:

1. What information must be stored?
2. Which operations are frequent?
3. Are updates allowed, or is the data static?
4. Are queries about one item, a prefix, or a range?
5. What ordering or correctness guarantees are required?
6. What are the expected input sizes and memory constraints?

For example, a workload that repeatedly asks for the sum of values in an interval has different needs from one that repeatedly searches words by prefix.

### Operation-focused selection guide

| Requirement | Structure to investigate |
|---|---|
| Find keys by a shared prefix | Trie |
| Update one array position and query an interval | Segment tree or Fenwick tree |
| Query a static range minimum or maximum | Sparse table |
| Maintain disjoint groups and test connectivity | Disjoint Set Union (DSU) |
| Keep only the most recently used cache entries | LRU cache |
| Understand storage indexes used by database systems | B-tree / B+ tree concepts |

This table is a starting point, not a universal performance guarantee. Input constraints, update patterns, implementation details, and the exact query all matter.

---

## 2. Tries (prefix trees)

A **trie** stores sequences—commonly strings—one element at a time. Each edge represents a character, and a path from the root represents a prefix.

A node needs to distinguish between:
- a prefix that exists because it leads to longer words;
- a complete stored word.

A terminal marker provides that distinction.

### Example

For the words `app`, `apple`, and `apt`, the paths for `app` and `apple` share the prefix `app`. The node after the second `p` is terminal for `app`, but it also has a child leading to `apple`.

### Basic Python implementation

This implementation uses dictionaries for child lookup. The terminal marker is stored separately from child characters.

```python
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

    def search(self, word):
        node = self.root

        for character in word:
            if character not in node.children:
                return False
            node = node.children[character]

        return node.is_word

    def starts_with(self, prefix):
        node = self.root

        for character in prefix:
            if character not in node.children:
                return False
            node = node.children[character]

        return True
```

### Trace the operations

Suppose the trie contains `deploy` and `deployment`.

- `starts_with("depl")` returns `True`.
- `search("deploy")` returns `True`.
- `search("deplo")` returns `False`, even though the path may exist as a prefix.
- `search("developer")` returns `False` if that word was not inserted.

### Complexity

Let (L) be the length of the input string.

| Operation | Typical complexity for this implementation |
|---|---:|
| Insert | (O(L)) expected |
| Search | (O(L)) expected |
| Prefix check | (O(L)) expected |

The expected-time description assumes average constant-time dictionary child lookup. Space depends on the number of distinct prefix nodes and the child-map representation. In the worst case, inserting a previously unseen word can add up to (L) nodes.

### Where tries are useful

- Autocomplete and prefix search.
- Dictionary and word-search problems.
- Routing or lookup tasks where keys share prefixes.
- Searching configuration keys by namespace or prefix in a teaching exercise.

A trie is not automatically the best choice for every string lookup. If the workload needs only exact-key lookup, a hash table may be simpler and more memory-efficient.

### Practice

1. Add a method that returns all words beginning with a prefix.
2. Add deletion while preserving nodes shared by other words.
3. Count how many inserted words begin with a given prefix.
4. Decide how the implementation should handle empty strings and repeated insertion.

---

## 3. Segment trees

A **segment tree** stores information about intervals of an array. It is useful when values can change and queries repeatedly ask about ranges.

Common range operations include:
- sum;
- minimum or maximum;
- greatest common divisor;
- other operations that can combine two adjacent interval results.

The operation must have a suitable identity and combination rule for the implementation being used.

### Core idea

For an array of length (n), the root represents the whole interval. Its children represent subintervals. The tree is built until each leaf represents one array element.

For a range-sum tree:
- each leaf stores one array value;
- each internal node stores the sum of its children;
- a range query combines the covered pieces;
- a point update changes one leaf and recalculates the nodes on its path to the root.

### Recursive range-sum implementation

The following version uses a 1-based tree array and 0-based input-array indices. It allocates `4 * n` slots for a simple recursive layout. Empty input is handled explicitly.

```python
class SegmentTree:
    def __init__(self, values):
        self.n = len(values)
        self.tree = [0] * (4 * self.n) if self.n else []
        if self.n:
            self._build(1, 0, self.n - 1, values)

    def _build(self, node, left, right, values):
        if left == right:
            self.tree[node] = values[left]
            return

        middle = (left + right) // 2
        self._build(node * 2, left, middle, values)
        self._build(node * 2 + 1, middle + 1, right, values)
        self.tree[node] = (
            self.tree[node * 2] + self.tree[node * 2 + 1]
        )

    def update(self, index, value):
        if not 0 <= index < self.n:
            raise IndexError("index out of range")
        self._update(1, 0, self.n - 1, index, value)

    def _update(self, node, left, right, index, value):
        if left == right:
            self.tree[node] = value
            return

        middle = (left + right) // 2
        if index <= middle:
            self._update(node * 2, left, middle, index, value)
        else:
            self._update(node * 2 + 1, middle + 1, right, index, value)

        self.tree[node] = (
            self.tree[node * 2] + self.tree[node * 2 + 1]
        )

    def query(self, query_left, query_right):
        if not 0 <= query_left <= query_right < self.n:
            raise IndexError("invalid query range")
        return self._query(
            1, 0, self.n - 1, query_left, query_right
        )

    def _query(self, node, left, right, query_left, query_right):
        if query_left <= left and right <= query_right:
            return self.tree[node]

        if right < query_left or query_right < left:
            return 0

        middle = (left + right) // 2
        left_sum = self._query(
            node * 2, left, middle, query_left, query_right
        )
        right_sum = self._query(
            node * 2 + 1, middle + 1, right, query_left, query_right
        )
        return left_sum + right_sum
```

Example:

```python
tree = SegmentTree([2, 4, 1, 7, 3])

print(tree.query(1, 3))  # 12: 4 + 1 + 7

tree.update(2, 5)
print(tree.query(1, 3))  # 16: 4 + 5 + 7
```

### Complexity

For this balanced recursive segment-tree layout:

| Operation | Complexity |
|---|---:|
| Build | (O(n)) |
| Point update | (O(\log n)) |
| Range query | (O(\log n)) |
| Storage | (O(n)) |

The query complexity assumes the range can be represented by a logarithmic number of tree segments and each segment result can be combined in constant time.

### Important design detail

The implementation above is specifically a **range-sum, point-assignment** tree. For minimum queries, the combine operation changes from addition to minimum, and the identity for a non-overlapping segment must change accordingly. More advanced operations may need different node state or lazy propagation.

### Lazy propagation preview

If a problem requires updating an entire interval—for example, adding a value to every element in a range—updating each leaf individually may be too expensive.

**Lazy propagation** stores pending updates at internal nodes and pushes them down only when a later operation needs more detail. The exact node metadata and update-combination rules depend on the operation.

Do not add lazy propagation until the basic build, point-update, and range-query invariants are clear.

---

## 4. Fenwick trees (Binary Indexed Trees)

A **Fenwick tree**, also called a **Binary Indexed Tree (BIT)**, supports prefix aggregation with a compact array representation.

It is especially useful for:
- point updates;
- prefix-sum queries;
- range-sum queries derived from two prefix sums.

This chapter uses addition and 1-based internal indexing.

### The low bit

For a positive integer `i`, the expression:

```python
i & -i
```

isolates its lowest set bit. A Fenwick tree uses this value to determine the size of the interval represented by an internal position.

### Implementation

```python
class FenwickTree:
    def __init__(self, size):
        if size < 0:
            raise ValueError("size must be non-negative")
        self.n = size
        self.tree = [0] * (size + 1)

    def add(self, index, delta):
        """Add delta to the value at 0-based index."""
        if not 0 <= index < self.n:
            raise IndexError("index out of range")

        i = index + 1
        while i <= self.n:
            self.tree[i] += delta
            i += i & -i

    def prefix_sum(self, end):
        """Return the sum over 0-based indices [0, end)."""
        if not 0 <= end <= self.n:
            raise IndexError("prefix endpoint out of range")

        total = 0
        i = end
        while i > 0:
            total += self.tree[i]
            i -= i & -i
        return total

    def range_sum(self, left, right):
        """Return the sum over 0-based indices [left, right)."""
        if not 0 <= left <= right <= self.n:
            raise IndexError("invalid range")
        return self.prefix_sum(right) - self.prefix_sum(left)
```

Example:

```python
bit = FenwickTree(5)

for index, value in enumerate([2, 4, 1, 7, 3]):
    bit.add(index, value)

print(bit.prefix_sum(4))   # 14
print(bit.range_sum(1, 4))  # 12

bit.add(2, 4)  # Increase index 2 from 1 to 5
print(bit.range_sum(1, 4))  # 16
```

### Complexity

| Operation | Complexity |
|---|---:|
| Point addition | (O(\log n)) |
| Prefix sum | (O(\log n)) |
| Range sum | (O(\log n)) |
| Storage | (O(n)) |

The range-sum method relies on subtraction of prefix sums. For other aggregate operations, this exact range-query technique may not apply.

### Fenwick tree vs segment tree

| Question | Fenwick tree | Segment tree |
|---|---|---|
| Prefix sum | Natural fit | Supported |
| Range sum | Difference of two prefixes | Direct range query |
| Point addition | Supported | Supported |
| Point assignment | Track old value and apply a delta | Directly supported by the example |
| More complex range operations | More limited | More flexible |
| Implementation size | Usually smaller | Usually larger |

Choose based on the operations required, not on the assumption that one structure is always faster.

---

## 5. Sparse tables

A **sparse table** is a static range-query structure. It preprocesses an array so that queries such as range minimum can be answered efficiently when the array does not change.

It is a good fit for idempotent operations such as:
- minimum;
- maximum;
- greatest common divisor.

An operation is idempotent when combining a value with itself does not change it. For example, `min(x, x) == x`.

### Preprocessing idea

- Level 0 stores intervals of length 1.
- Level 1 stores results for intervals of length 2.
- Level 2 stores results for intervals of length 4.
- In general, level (k) stores results for intervals of length (2^k).

A query interval can be covered by two overlapping blocks of the largest power-of-two length that fits. For an idempotent operation such as minimum, the overlap does not change the answer.

### Range-minimum implementation

This implementation uses half-open query ranges `[left, right)`.

```python
class SparseTableMin:
    def __init__(self, values):
        self.n = len(values)
        self.logs = [0] * (self.n + 1)

        for i in range(2, self.n + 1):
            self.logs[i] = self.logs[i // 2] + 1

        self.table = []
        if not self.n:
            return

        self.table.append(list(values))
        level = 1

        while (1 << level) <= self.n:
            half = 1 << (level - 1)
            width = 1 << level
            previous = self.table[level - 1]

            current = [
                min(previous[i], previous[i + half])
                for i in range(self.n - width + 1)
            ]
            self.table.append(current)
            level += 1

    def query(self, left, right):
        """Return the minimum over the non-empty range [left, right)."""
        if not 0 <= left < right <= self.n:
            raise IndexError("invalid or empty query range")

        length = right - left
        level = self.logs[length]
        block_length = 1 << level

        return min(
            self.table[level][left],
            self.table[level][right - block_length]
        )
```

Example:

```python
values = [8, 3, 6, 2, 9, 1, 7]
table = SparseTableMin(values)

print(table.query(1, 5))  # 2: minimum of [3, 6, 2, 9]
print(table.query(4, 7))  # 1: minimum of [9, 1, 7]
```

### Complexity

| Operation | Complexity |
|---|---:|
| Preprocessing | (O(n \log n)) |
| Query for minimum | (O(1)) |
| Storage | (O(n \log n)) |
| Point update | Not supported efficiently by this static design |

The constant-time query relies on an idempotent operation. Do not copy this query formula unchanged for sum: overlapping blocks would count some values twice.

---

## 6. Disjoint Set Union (DSU)

A **Disjoint Set Union** structure maintains a collection of non-overlapping sets. It supports two central operations:

- `find(x)`: identify the representative of the set containing `x`;
- `union(a, b)`: merge the sets containing `a` and `b`.

DSU is useful for connectivity questions, grouping, and Kruskal's minimum spanning tree algorithm.

This structure was introduced in the Graphs and Advanced Graph Algorithms chapters. Here, focus on the data-structure trade-off and implementation invariant.

### Path compression and union by size

```python
class DisjointSetUnion:
    def __init__(self, size):
        self.parent = list(range(size))
        self.component_size = [1] * size

    def find(self, item):
        if not 0 <= item < len(self.parent):
            raise IndexError("item out of range")

        if self.parent[item] != item:
            self.parent[item] = self.find(self.parent[item])

        return self.parent[item]

    def union(self, first, second):
        root_a = self.find(first)
        root_b = self.find(second)

        if root_a == root_b:
            return False

        if self.component_size[root_a] < self.component_size[root_b]:
            root_a, root_b = root_b, root_a

        self.parent[root_b] = root_a
        self.component_size[root_a] += self.component_size[root_b]
        return True
```

With path compression and union by size, a sequence of operations has amortized time (O(\alpha(n))) per operation, where (alpha) is the inverse Ackermann function. For practical input sizes this grows extremely slowly, but it is not mathematically identical to constant time.

See [11 — Graphs](../11%20Graphs/README.md) and [15 — Advanced Graph Algorithms](../15%20Advanced%20Graph%20Algorithms/README.md) for graph applications.

---

## 7. LRU cache

An **LRU (Least Recently Used) cache** evicts the item that has gone unused for the longest time when it needs to make room for another item.

The key operations are:
- retrieve an item and mark it recently used;
- insert or update an item and mark it recently used;
- evict the least recently used item when capacity is exceeded.

Python's `collections.OrderedDict` provides operations that can be used to build a simple LRU cache. This example is an educational implementation, not a complete production cache.

```python
from collections import OrderedDict


class LRUCache:
    def __init__(self, capacity):
        if capacity < 0:
            raise ValueError("capacity must be non-negative")

        self.capacity = capacity
        self.items = OrderedDict()

    def get(self, key):
        if key not in self.items:
            return None

        self.items.move_to_end(key)
        return self.items[key]

    def put(self, key, value):
        if self.capacity == 0:
            return

        if key in self.items:
            self.items.move_to_end(key)

        self.items[key] = value

        if len(self.items) > self.capacity:
            self.items.popitem(last=False)
```

For this implementation, dictionary-backed lookup and the relevant `OrderedDict` operations are expected/amortized constant time under the standard hash-table assumptions.

Design questions to consider:
- How should a cached `None` value be distinguished from a cache miss?
- Should the cache support expiration?
- Is it safe for concurrent callers?
- What should happen when capacity is zero?
- Is eviction based on item count or memory usage?

A real production cache may need synchronization, metrics, expiration, size limits, and explicit failure behavior.

---

## 8. B-trees and B+ trees: the database connection

A **B-tree** is a balanced search-tree family designed to keep data ordered while reducing the height of the tree through nodes that can contain multiple keys and children.

A **B+ tree** is a related design commonly used for database and storage indexes. In a typical B+ tree design:
- internal nodes guide searches;
- records or record references are stored in leaves;
- leaf nodes are often linked to support ordered/range traversal.

The exact page layout, fan-out, split/merge policy, and storage behavior depend on the database or storage engine. Do not assume every implementation has identical details.

### Why engineers should know the idea

When a database query filters or orders by an indexed key, the database may use an index structure to avoid scanning every row. Understanding tree indexes helps explain:
- why index key order matters;
- why range queries can benefit from ordered indexes;
- why inserts and deletes may require index maintenance;
- why indexes consume storage and add write overhead.

This is a conceptual introduction, not a claim about the internal implementation of any particular database. Consult the official documentation for the database you use before making product-specific statements.

---

## 9. Comparing the structures

| Structure | Strength | Important limitation |
|---|---|---|
| Trie | Prefix-oriented string lookup | Can use substantial memory |
| Segment tree | Flexible range queries with updates | More implementation complexity |
| Fenwick tree | Compact prefix aggregates and point updates | Supports a narrower set of operations |
| Sparse table | Fast static idempotent range queries | Not designed for updates |
| DSU | Dynamic connectivity under set merges | Does not support splitting a set |
| LRU cache | Bounded recency-based caching | Eviction policy may not fit every workload |
| B-tree / B+ tree | Ordered storage indexes and range access | Implementation details are storage-engine specific |

A useful interview answer explains both why a structure fits and why alternatives do not.

---

## 10. Common mistakes

1. **Choosing by name instead of workload:** list the required operations first.
2. **Ignoring indexing conventions:** segment trees often use inclusive intervals in examples; Fenwick and sparse-table implementations here use half-open ranges for public queries.
3. **Forgetting identity values:** a range-minimum tree needs a different no-overlap result from a range-sum tree.
4. **Using sparse-table overlap for non-idempotent operations:** the two-block query described here is for idempotent operations such as minimum.
5. **Assuming all updates are supported:** the sparse table here is static.
6. **Confusing a prefix with a complete word:** a trie needs a terminal marker.
7. **Calling amortized time constant:** state the assumptions and the actual amortized bound where relevant.
8. **Ignoring memory:** a structure can improve query time while increasing memory use.
9. **Skipping boundary tests:** empty input, one element, first/last index, and invalid ranges are frequent sources of defects.
10. **Treating a teaching cache as production-ready:** concurrency, expiration, and memory accounting are separate design requirements.

---

## 11. Testing checklist

For every implementation, test:

- empty input where supported;
- one element;
- repeated values or repeated keys;
- first and last valid positions;
- full-range and single-element queries;
- updates followed by queries;
- invalid indices and ranges;
- values that are negative, zero, and positive where the operation permits them;
- repeated operations that should not change the result.

For range structures, compare randomized operations against a simple list-based reference implementation. A slow but straightforward reference is useful for checking a more complex implementation.

---

## 12. Practice progression

### Beginner

1. Implement a trie with insert, exact search, and prefix search.
2. Implement a Fenwick tree for prefix sums.
3. Implement a segment tree for range sums and point assignments.
4. Explain the difference between a static sparse table and an update-capable segment tree.

### Intermediate

5. Add word collection and deletion to a trie.
6. Extend a segment tree to range minimum.
7. Implement a sparse table for range maximum.
8. Implement an LRU cache and test eviction order.
9. Use DSU to count connected components as edges are added.

### Advanced

10. Add lazy propagation for range addition and range-sum queries.
11. Compare Fenwick and segment trees on the same operation sequence.
12. Investigate how a B+ tree supports ordered range access.
13. Write property-based or randomized tests against simple reference implementations.

---

## 13. Mini project — Range Metrics Index

**Goal:** Build a command-line or Python module that maintains a numeric sequence and answers range queries.

### Required features

- Load a sequence of integers.
- Support point updates.
- Answer range-sum queries.
- Answer range-minimum queries.
- Reject invalid indices and ranges with clear errors.
- Include tests that compare answers against direct list operations.

### Engineering tasks

1. Implement the first version with a plain list.
2. Record the cost of the operations conceptually before optimizing.
3. Implement a Fenwick tree for sums.
4. Implement a segment tree for sums and minimums.
5. Use the same deterministic operation sequence to verify both implementations.
6. Document the index convention and complexity assumptions.
7. Explain when the extra structure is worthwhile and when the plain list is simpler.

**Repository design decision:** Keep this project focused on correctness and data-structure trade-offs. Do not present it as a production monitoring or database engine.

---

## 14. Interview questions

1. When would you use a trie instead of a hash table?
2. Why does a trie need a terminal marker?
3. What makes a segment tree suitable for dynamic range queries?
4. What is the difference between a Fenwick tree and a segment tree?
5. Why can a sparse table answer a static range-minimum query in constant time?
6. Why does the two-overlapping-block sparse-table query require an idempotent operation?
7. What does the low bit represent in a Fenwick tree?
8. What do path compression and union by size achieve in DSU?
9. Why can an LRU cache use an ordered mapping?
10. What is the difference between a B-tree and a binary search tree at a high level?
11. Why do database indexes introduce write and storage costs?
12. How would you test a range-query implementation against a simple reference?

When answering, include the invariant, operation costs, assumptions, and at least one limitation.

---

## 15. Completion checklist

- [ ] I can explain when a trie is useful.
- [ ] I can implement insert, search, and prefix search in a trie.
- [ ] I can explain segment-tree interval decomposition.
- [ ] I can implement point updates and range queries in a segment tree.
- [ ] I can explain Fenwick tree indexing and the low-bit operation.
- [ ] I can derive a range sum from two prefix sums.
- [ ] I can explain why the sparse table in this chapter is static.
- [ ] I can state why its two-block query works for minimum and maximum.
- [ ] I can implement DSU with path compression and union by size.
- [ ] I can explain LRU eviction order.
- [ ] I understand the role of B-tree-family indexes at a conceptual level.
- [ ] I can test advanced structures against simple reference implementations.
- [ ] I can justify a data-structure choice from the workload.

---

## 16. Sources and quality notes

The following official Python documentation supports the standard-library APIs used in this chapter:

- [Python — `collections`](https://docs.python.org/3/library/collections.html) — includes `OrderedDict`, used in the LRU cache example.
- [Python — `functools`](https://docs.python.org/3/library/functools.html) — useful standard-library caching context.
- [Python — `bisect`](https://docs.python.org/3/library/bisect.html) — standard-library support for maintaining sorted sequences.
- [Python — `heapq`](https://docs.python.org/3/library/heapq.html) — standard-library heap operations, relevant to priority-based structures.

These references document Python APIs. They are not presented as proofs of the theoretical complexity of every data structure described above. The algorithmic complexity statements in this chapter assume the representations and operation models stated alongside them.

**Verified fact:** The cited Python documentation describes the named standard-library modules and APIs.

**Repository design decision:** The examples use readable implementations and explicit index conventions to make the underlying invariants easier to inspect.

**Demonstration:** The trie, segment tree, Fenwick tree, sparse table, DSU, and LRU cache code are educational implementations. They should be tested and adapted before use in a production system.

---

**Previous:** [15 — Advanced Graph Algorithms](../15%20Advanced%20Graph%20Algorithms/README.md)  
**Next:** [17 — Interview Patterns](../17%20Interview%20Patterns/README.md)
