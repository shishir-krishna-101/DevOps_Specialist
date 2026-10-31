# 09 — Trees and Binary Search Trees

Trees are hierarchical data structures. They are useful when information naturally has parent-child relationships, such as file systems, organizational structures, syntax trees, and dependency hierarchies.

> **Learning goal:** Understand tree terminology, traversal, recursive reasoning, binary trees, binary search trees, ordering invariants, complexity, and the situations where a tree is useful.

---

## 1. What Is a Tree?

A tree is a collection of nodes connected by edges with a hierarchical structure.

A simple tree can look like:

```text
        A
       / \
      B   C
     / \
    D   E
```

Here:

- `A` is the root.
- `B` and `C` are children of `A`.
- `D` and `E` are children of `B`.
- `B` is the parent of `D` and `E`.
- `D` and `E` are leaves because they have no children.

Trees are different from arrays and linked lists because the structure can branch.

---

## 2. Tree Terminology

### Root

The top node of the tree.

### Parent

A node directly above another node.

### Child

A node directly below another node.

### Sibling

Nodes that share the same parent.

### Leaf

A node with no children.

### Internal node

A node with at least one child.

### Edge

A connection between two nodes.

### Depth

The number of edges from the root to a node, under the usual convention.

### Height

The number of edges on a longest downward path from a node to a leaf, under the same convention.

> **Important:** Some textbooks define height and depth using node counts instead of edge counts. State the convention when precision matters.

---

## 3. Why Trees Matter

Trees appear whenever data has hierarchical relationships.

Examples include:

- directory structures
- organizational hierarchies
- dependency structures
- expression trees
- search trees
- decision trees
- parsers and syntax representations

In DevOps and infrastructure work, tree-like reasoning can help when exploring configuration hierarchies, dependency relationships, or nested resource structures.

These examples describe possible applications of tree reasoning; they are not claims that a particular external tool internally uses a specific tree implementation.

---

## 4. Binary Trees

A **binary tree** is a tree in which each node has at most two children.

The children are commonly called:

```text
left child
right child
```

Example:

```text
          10
         /  \
        5    15
       / \     \
      2   7     20
```

A binary tree does **not** automatically mean that values are sorted.

That distinction is important:

> **Binary tree describes the number of children. Binary search tree describes an ordering rule.**

---

## 5. Representing a Tree

A common representation uses a node object containing:

```text
value
left
right
```

Conceptually:

```python
class TreeNode:
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None
```

A tree can then be connected manually:

```python
root = TreeNode(10)
root.left = TreeNode(5)
root.right = TreeNode(15)
```

The exact class design is a repository learning implementation, not a requirement imposed by Python.

---

## 6. Tree Traversal

Traversal means visiting nodes according to a defined order.

The foundational depth-first traversals are:

1. preorder
2. inorder
3. postorder

A breadth-first traversal is commonly called level-order traversal.

Traversal is one of the most important tree skills because many tree problems reduce to:

```text
visit every relevant node
maintain some state
return the required result
```

---

## 7. Preorder Traversal

Preorder visits:

```text
Node → Left → Right
```

For:

```text
        A
       / \
      B   C
     / \
    D   E
```

preorder produces:

```text
A B D E C
```

Preorder is useful when the parent needs to be processed before its descendants.

Conceptual recursive implementation:

```python
def preorder(node):
    if node is None:
        return

    print(node.value)
    preorder(node.left)
    preorder(node.right)
```

---

## 8. Inorder Traversal

Inorder visits:

```text
Left → Node → Right
```

For the same tree:

```text
        A
       / \
      B   C
     / \
    D   E
```

inorder produces:

```text
D B E A C
```

Inorder becomes especially important for binary search trees because, under the standard BST ordering invariant, an inorder traversal visits values in sorted order.

---

## 9. Postorder Traversal

Postorder visits:

```text
Left → Right → Node
```

For the same tree:

```text
        A
       / \
      B   C
     / \
    D   E
```

postorder produces:

```text
D E B C A
```

Postorder is useful when children must be processed before their parent.

A classic example is computing information from child subtrees before computing the corresponding information for the current node.

---

## 10. Level-Order Traversal

Level-order traversal visits nodes breadth-first.

For:

```text
        A
       / \
      B   C
     / \
    D   E
```

level order is:

```text
A B C D E
```

A queue is the natural data structure for this traversal:

```text
put root into queue

while queue is not empty:
    remove next node
    process it
    add its children
```

This connects tree traversal directly to the stack and queue concepts from the previous chapters.

---

## 11. Recursive Tree Reasoning

Trees are naturally recursive because each child can itself be the root of a smaller subtree.

For example:

```text
        A
       / \
      B   C
```

can be viewed as:

```text
Tree(A)
├── Tree(B)
└── Tree(C)
```

A useful mental model is:

> Solve the problem for the current node using the answers returned by its child subtrees.

This idea appears repeatedly in tree algorithms.

---

## 12. Tree Height

A recursive definition of height under the edge-count convention is:

```text
height(empty tree) = -1
height(leaf) = 0
height(node) = 1 + max(height(left), height(right))
```

A corresponding implementation can be:

```python
def height(node):
    if node is None:
        return -1

    return 1 + max(height(node.left), height(node.right))
```

For a tree with `n` nodes, computing height by visiting every node takes **O(n)** time.

The recursion depth depends on the tree's height.

---

## 13. Counting Nodes

Counting nodes is another simple recursive pattern:

```text
count(node)
= 1
  + count(left subtree)
  + count(right subtree)
```

Base case:

```text
count(None) = 0
```

The algorithm visits every node once, so its time complexity is **O(n)** and its auxiliary call-stack usage is proportional to the tree height for the recursive implementation.

---

## 14. Binary Search Trees

A **binary search tree (BST)** is a binary tree with an ordering invariant.

A common strict-ordering convention is:

```text
values in left subtree < node value < values in right subtree
```

For example:

```text
          10
         /  \
        5    15
       / \     \
      2   7     20
```

This ordering lets comparisons guide the search.

> **Duplicate handling is a design choice unless the specific BST definition or implementation specifies a rule.**

Do not silently assume whether duplicates go left, right, or are rejected.

---

## 15. Searching a BST

To search for a target:

```text
compare target with current node

if equal:
    found

if target is smaller:
    search left subtree

if target is larger:
    search right subtree
```

For the example tree, searching for `7` follows:

```text
10 → 5 → 7
```

The ordering invariant lets us discard an entire subtree at each comparison.

---

## 16. BST Search Complexity

The running time depends on the tree height `h`.

A search follows at most one root-to-leaf path, so the time is:

```text
O(h)
```

If the tree is reasonably balanced:

```text
h = O(log n)
```

and search can therefore be **O(log n)**.

But a BST does not automatically remain balanced.

A highly skewed tree can look like:

```text
10
  \
   15
     \
      20
        \
         25
```

Then:

```text
h = O(n)
```

and search becomes **O(n)**.

This is one of the most important BST lessons:

> **BST search complexity depends on height, not simply on the fact that the structure is a BST.**

---

## 17. BST Insertion

Insertion follows the same ordering rule as search.

For a new value:

1. Start at the root.
2. Compare the value with the current node.
3. Move left or right according to the ordering rule.
4. Continue until the appropriate empty child position is reached.
5. Insert the new node there.

The running time is **O(h)**.

Therefore, balanced and skewed trees can have very different practical performance.

---

## 18. BST Deletion

Deletion has three important structural cases.

### Case 1 — Leaf

The node has no children.

Remove it directly.

### Case 2 — One child

Replace the node with its only child.

### Case 3 — Two children

The node has both left and right subtrees.

A common strategy is to replace its value with an appropriate neighboring value according to the BST ordering, such as the inorder successor, and then remove that successor from its original position.

The exact implementation must preserve the BST invariant.

Deletion takes **O(h)** under the standard pointer-based BST model.

---

## 19. Inorder Successor

For a node with a right subtree, its inorder successor is the smallest value in that right subtree under the usual BST ordering.

For example:

```text
        10
       /  \
      5    15
          /  \
         12   20
```

The successor of `10` is `12`.

The successor concept is useful for deletion and ordered traversal problems.

---

## 20. BST Validation

A common interview problem asks:

> Is this binary tree a valid BST?

Checking only the immediate children is not enough.

For example:

```text
        10
       /  \
      5    15
          /
         7
```

The `7` node is in the right subtree of `10`, so under the strict ordering convention it violates the rule even though `7 < 15`.

A correct validation approach carries an allowed range or equivalent ordering constraints down the tree.

Conceptually:

```text
root must be within (-∞, +∞)

left child must be within (-∞, root)
right child must be within (root, +∞)
```

The exact boundary rules must be adjusted if the BST allows duplicates.

---

## 21. Kth Smallest Element in a BST

Because inorder traversal of a valid BST produces values in sorted order, the kth visited node during inorder traversal is the kth smallest value under the tree's ordering convention.

Conceptual process:

```text
inorder traversal
      ↓
count visited nodes
      ↓
stop at k
```

This is a good example of using a structural invariant to simplify a problem.

---

## 22. Lowest Common Ancestor

The **lowest common ancestor (LCA)** of two nodes is the deepest node that is an ancestor of both.

In a general binary tree, finding the LCA may require exploring both subtrees.

In a BST, the ordering invariant can provide a shortcut.

If both target values are smaller than the current node, move left.

If both are larger, move right.

Otherwise, the current node is the split point and therefore the LCA under the usual assumptions.

---

## 23. Balanced vs Unbalanced Trees

A balanced tree keeps its height relatively small compared with the number of nodes.

A highly skewed tree can behave much more like a linked list.

Conceptually:

```text
Balanced:
        8
      /   \
     4     12
    / \   /  \
   2   6 10  14
```

versus:

```text
Skewed:
8
 \
  10
    \
     12
       \
        14
```

The balanced version has much smaller height, which can significantly improve operations whose cost is proportional to height.

---

## 24. Why Self-Balancing Trees Exist

A plain BST does not guarantee logarithmic height.

Self-balancing tree families modify the structure as updates occur so that the height remains bounded more tightly.

Examples include:

- AVL trees
- Red-Black trees

These are advanced data structures and are introduced here to explain why balancing matters. Their detailed invariants and rotations belong in the advanced data-structures section of this roadmap.

Do not treat every BST as a self-balancing tree.

---

## 25. Tree Traversal Complexity

For a tree containing `n` nodes, a traversal that visits every node once takes:

```text
Time: O(n)
```

Additional space depends on the implementation.

For recursive DFS:

```text
Auxiliary call stack: O(h)
```

where `h` is the tree height.

For iterative level-order traversal:

```text
Queue space: O(w)
```

where `w` is the maximum width of the tree.

In the worst case, these can be proportional to `n`.

---

## 26. DFS vs BFS on Trees

### DFS

Depth-first traversal can be implemented recursively or with an explicit stack.

Useful when:

- subtree results matter
- path-based reasoning is needed
- recursive structure is natural
- depth should be explored before siblings

### BFS

Breadth-first traversal uses a queue.

Useful when:

- processing by depth matters
- the nearest level is important
- level-by-level output is required

Neither is universally better. Choose based on the problem's structure.

---

## 27. Path-Based Problems

Many tree interview problems ask about paths.

Examples:

- maximum depth
- minimum depth
- root-to-leaf paths
- path sums
- longest path
- whether a target path exists

A useful pattern is to carry state while traversing:

```text
current node
current path/state
child subtrees
```

When returning from a recursive call, make sure the state is restored if mutable shared state is being used.

This connects tree problems with the backtracking concepts from Chapter 06.

---

## 28. Common Tree Pitfalls

### Mistake 1 — Confusing binary tree with BST

A binary tree does not automatically have sorted values.

### Mistake 2 — Assuming BST operations are always O(log n)

They are O(h), and `h` can be `O(n)`.

### Mistake 3 — Incorrect base cases

Empty subtrees must be handled explicitly.

### Mistake 4 — Losing subtree information

When recursively returning a value, define exactly what the return value represents.

### Mistake 5 — Incorrect duplicate handling

State the duplicate policy.

### Mistake 6 — Mixing traversal orders

Write the order explicitly before coding:

```text
preorder  = N L R
inorder   = L N R
postorder = L R N
```

### Mistake 7 — Ignoring recursion depth

A skewed tree can produce recursion depth proportional to `n` in recursive implementations.

---

## 29. A Repeatable Tree-Problem Process

When solving a tree problem:

```text
1. Identify the tree type
2. Define the node representation
3. Write the required invariant
4. Choose DFS or BFS
5. Define the base case
6. Define what the recursive call returns
7. Combine child results
8. Trace a tiny tree
9. Test empty and single-node cases
10. Analyze time and space
```

For BST problems, add:

```text
11. Verify the ordering invariant
12. State the duplicate policy
13. Express complexity using tree height
```

---

## 30. Practice Progression

### Beginner

1. Count nodes.
2. Find tree height.
3. Implement preorder traversal.
4. Implement inorder traversal.
5. Implement postorder traversal.
6. Implement level-order traversal.

### Intermediate

7. Search a BST.
8. Insert into a BST.
9. Find minimum and maximum.
10. Validate a BST.
11. Find kth smallest value.
12. Find lowest common ancestor.

### Advanced

13. Delete from a BST.
14. Serialize and deserialize a tree.
15. Find tree diameter.
16. Solve path-sum variants.
17. Convert between tree representations.
18. Compare recursive and iterative traversal strategies.

---

## 31. Mini Project — Hierarchical Resource Explorer

### Goal

Build a small learning project that represents a hierarchy of resources as a tree.

Example repository-level model:

```text
environment
├── services
│   ├── api
│   └── worker
└── databases
    └── primary
```

### Features

Implement:

- add a node
- display the hierarchy
- preorder traversal
- search by name
- calculate depth
- count descendants

### Learning objectives

This project reinforces:

- tree representation
- recursion
- traversal
- search
- state propagation
- complexity analysis

> **Demonstration note:** This hierarchy is intentionally simplified for learning. It is a repository design example, not a claim about how any particular cloud or orchestration platform stores resources internally.

---

## 32. Interview Questions

### Fundamentals

1. What is the difference between a tree and a graph?
2. What is the difference between a binary tree and a BST?
3. Define depth and height.
4. What is a leaf node?
5. Compare preorder, inorder, and postorder traversal.

### BST

6. Why can BST search be `O(n)`?
7. When is BST search `O(log n)`?
8. How do you insert into a BST?
9. Explain the three BST deletion cases.
10. How do you validate a BST correctly?
11. Why does inorder traversal of a valid BST produce sorted values?
12. How can you find the kth smallest value?

### Problem solving

13. When would BFS be preferable to DFS on a tree?
14. How does tree height affect recursive space usage?
15. How would you find the lowest common ancestor?
16. How would you calculate tree diameter?
17. What changes when duplicate values are allowed?

---

## 33. Completion Checklist

Before moving on, you should be able to:

- [ ] Explain tree terminology.
- [ ] Distinguish binary trees from BSTs.
- [ ] Implement preorder traversal.
- [ ] Implement inorder traversal.
- [ ] Implement postorder traversal.
- [ ] Implement level-order traversal.
- [ ] Explain why recursion fits trees naturally.
- [ ] Calculate tree height.
- [ ] Search a BST.
- [ ] Insert into a BST.
- [ ] Explain BST deletion cases.
- [ ] Validate a BST using global constraints.
- [ ] Explain why BST complexity depends on height.
- [ ] Recognize balanced vs skewed trees.
- [ ] Choose DFS or BFS based on the problem.
- [ ] Analyze tree time and space complexity.

---

## 34. Sources and Quality Notes

This chapter separates general algorithmic concepts from Python-specific implementation details.

### Python-specific references

- Python tutorial — Classes: https://docs.python.org/3/tutorial/classes.html
- Python standard types: https://docs.python.org/3/library/stdtypes.html

### Quality notes

- Complexity statements are for the stated standard implementations and assumptions.
- BST performance is expressed in terms of height before giving balanced-tree implications.
- Duplicate-value behavior is explicitly treated as an implementation/design choice unless a definition specifies otherwise.
- Tree terminology can vary slightly between sources, especially for height conventions.
- Repository examples are demonstrations and should not be presented as official implementation details of external DevOps platforms.

---

## Next

Continue to:

**[10 — Heaps and Priority Queues](../10%20Heaps%20and%20Priority%20Queues/README.md)**

← [08 — Searching](../08%20Searching/README.md)
