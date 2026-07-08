---
title: Binary Trees — DFS & BFS
topic: dsa
---


> Not a reference. A story.
> A tree is what you reach for the moment "a straight line of nodes" stops being
> enough — the moment your data has *hierarchy*. Read this to understand why the
> traversals are named the way they are, why almost every tree problem is the
> same three lines of recursion, and why a search tree can quietly rot into a
> linked list. Then the LeetCode problems stop looking like 40 different puzzles
> and start looking like one puzzle wearing 40 hats.

See also [data-structures.md](data-structures.md) for the array/list/heap
foundations these build on, [recursion.md](recursion.md) for the call-stack
mental model behind the "three lines" every traversal uses, and
[patterns.md](patterns.md) for the DFS/BFS patterns in context. There's an
interactive companion —
[the Binary Trees guide](../../guides/binary-trees.html) — that steps through the
four traversal orders and BST search visually.

---

## Why Trees Exist

Walk back through the [data structures story](data-structures.md). The array
gave you instant access but a frozen shape. The linked list freed the shape —
insert anywhere in O(1) — but only along a single line: each node points to *the
next one*. One neighbour. A straight chain.

That's fine until your data isn't a line. A file system isn't a line — a folder
contains folders. An org chart isn't a line — a manager has reports who have
reports. An HTML page isn't a line — a `<div>` wraps other elements. All of these
are the same shape: **one thing owns several things, each of which owns several
more.** Hierarchy.

A tree is the linked list's answer to hierarchy: *let a node point to more than
one next node.* That single change — from "one child" to "many children" — is the
entire idea. Everything else is constraints layered on top.

---

## The Three Layers: Tree → Binary → Search

The most common beginner confusion — *"does the left child have to be smaller
than the right?"* — is really a question about **which layer you're standing on.**
There are three, stacked, and each adds exactly one rule.

### Layer 1 — Tree (the base definition)

A tree is a connected graph with **no cycles**. Concretely:

- Every node has exactly one parent — except the **root**, which has none.
- There is exactly one path between any two nodes.
- N nodes always means **N − 1 edges** (every node has one parent-edge except
  the root).

That's the whole definition. Nothing about ordering. Nothing about how many
children a node may have. A file system, an org chart, a DOM — all trees in this
general sense. The "no cycles, one parent" rule is what makes recursion safe:
you can walk into a subtree knowing you'll never loop back to where you started.

### Layer 2 — Binary Tree (+ one rule)

Add a single constraint: **every node has at most 2 children.** We call them
`left` and `right` by convention. That's it. Notice what's *still* missing —
nothing says which value goes where. A binary tree with `8` on the left and `2`
on the right breaks no rule at all.

In Java, the whole structure is this one class:

```java
class TreeNode {
    int val;
    TreeNode left;   // at most one left child (may be null)
    TreeNode right;  // at most one right child (may be null)

    TreeNode(int val) { this.val = val; }
}
```

A `null` child is how you say "no child here." A **leaf** is a node whose `left`
and `right` are both `null`.

### Layer 3 — Binary Search Tree (+ one more rule)

*Now* comes the ordering rule, and it's what people mistakenly attribute to plain
binary trees. A **BST** is a binary tree plus an **invariant** enforced at every
node, recursively:

> For any node: everything in its **left** subtree is smaller, everything in its
> **right** subtree is larger.

Not just its direct children — its *entire* left and right subtrees. This is the
rule that makes "search" in the name earn its keep (more on that below).

### The Mental Model

> "Tree" is the shape. "Binary" caps the branching at 2. "Search" orders the
> values. When a problem says **binary tree**, assume nothing about order. Only
> when it says **BST** are you allowed to lean on left-&lt;-right.

---

## Traversals: One Tree, Four Visiting Orders

Once you have a tree, the first question is always: *in what order do I visit the
nodes?* There are four canonical answers, and three of them are secretly the
same algorithm.

Take this tree for every example below:

```
        1
       / \
      2   3
     /
    4
```

The three DFS orders are named for **where the root falls relative to its two
subtrees** — that is the entire trick:

| Name | "means" | Order | On the tree above |
|------|---------|-------|-------------------|
| **Pre**order | root **before** subtrees | Root, Left, Right | `1, 2, 4, 3` |
| **In**order | root **in between** | Left, Root, Right | `4, 2, 1, 3` |
| **Post**order | root **after** subtrees | Left, Right, Root | `4, 2, 3, 1` |

Left always comes before right in all three — that never changes. The name only
tells you **when the root gets processed** relative to that. One-line mnemonic:
**pre / in / post = before / middle / after**, and that's literally where "Root"
sits in the sequence "Left … Right."

Here's the punchline that makes this cheap to remember — **all three are the same
recursion**. You only move the "process this node" line:

```java
void dfs(TreeNode node) {
    if (node == null) return;   // base case: fell off the tree

    // process(node);           ← preorder: process HERE

    dfs(node.left);

    // process(node);           ← inorder: process HERE

    dfs(node.right);

    // process(node);           ← postorder: process HERE
}
```

Concretely, preorder that collects values into a list:

```java
List<Integer> preorder(TreeNode root) {
    List<Integer> out = new ArrayList<>();
    walk(root, out);
    return out;
}
void walk(TreeNode node, List<Integer> out) {
    if (node == null) return;
    out.add(node.val);      // root
    walk(node.left, out);   // left
    walk(node.right, out);  // right
}
```

Move `out.add(node.val)` between the two recursive calls and you have inorder;
move it after and you have postorder. Same six lines.

> **Why inorder is special for a BST:** because a BST puts smaller on the left
> and larger on the right, "Left, Root, Right" visits values in **sorted order**.
> Inorder traversal of a BST = the sorted list, for free. That single fact solves
> *Kth Smallest Element in a BST*, *Validate BST*, and more.

### DFS without recursion (the stack version)

Recursion uses the *call stack* under the hood. You can make that stack explicit
with a `Deque` — interviewers love asking for this to check you understand what
recursion actually does:

```java
List<Integer> preorderIterative(TreeNode root) {
    List<Integer> out = new ArrayList<>();
    if (root == null) return out;

    Deque<TreeNode> stack = new ArrayDeque<>();
    stack.push(root);

    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();
        out.add(node.val);
        // push right FIRST so left is popped first (LIFO)
        if (node.right != null) stack.push(node.right);
        if (node.left  != null) stack.push(node.left);
    }
    return out;
}
```

### Level order (BFS) — the odd one out

Level order is **not** part of the pre/in/post family. It's named for what it
does: visit the tree **row by row**, top to bottom, ignoring the left/right depth
dance entirely. On our tree that's `1` → `2, 3` → `4`.

It's a different algorithm — **BFS with a queue**, not recursion. The queue holds
"nodes I've discovered but not yet processed," and because a queue is FIFO, you
drain each level before reaching the next:

```java
List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> levels = new ArrayList<>();
    if (root == null) return levels;

    Queue<TreeNode> queue = new ArrayDeque<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int size = queue.size();          // how many nodes on THIS level
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {  // drain exactly this level
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left  != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        levels.add(level);
    }
    return levels;
}
```

That `int size = queue.size()` snapshot at the top of each loop is the trick that
separates the levels — it freezes the current row's width before you start adding
the next row's children. Master this one skeleton and *Right Side View*, *Zigzag
Level Order*, and *Average of Levels* all fall out of it with a tweak.

### The Mental Model

> Stack → depth. Queue → breadth. DFS plunges down one path to the bottom before
> backtracking; BFS fans out level by level. **Recursion is just DFS with the
> stack hidden inside the language.**

---

## The Pattern That Unlocks the "Hard" Problems

Beginners write DFS that only *pushes* information down (preorder) — "here's the
path so far." The leap to intermediate is DFS that **returns information back up**
(postorder). The shape is always the same:

```java
int solve(TreeNode node) {
    if (node == null) return /* identity value, e.g. 0 or -1 */;

    int left  = solve(node.left);    // answer for the left subtree
    int right = solve(node.right);   // answer for the right subtree

    return /* combine left, right, and node into this subtree's answer */;
}
```

You compute children first, then combine — that's postorder. **Height** is the
canonical example:

```java
int maxDepth(TreeNode node) {
    if (node == null) return 0;
    return 1 + Math.max(maxDepth(node.left), maxDepth(node.right));
}
```

"My height = 1 + the taller of my two children's heights." Once you see height
this way, a whole family of problems is just height with bookkeeping:

- **Diameter** — longest path between any two nodes. At each node, the path
  *through* it is `leftHeight + rightHeight`; track the max as a side effect
  while you compute height.
- **Balanced Binary Tree** — a tree where no node's two subtrees differ in height
  by more than 1. Return height, but return a sentinel (`-1`) the instant you
  detect imbalance, so it short-circuits up the whole tree.
- **Maximum Path Sum** — same pattern, but the value flowing up is a running sum,
  and you discard negative subtree contributions.

> If a problem's answer at a node depends on its children's answers, reach for
> **postorder-with-a-return-value**. This single pattern is most of the "medium"
> and "hard" tree tier.

---

## Binary Search Trees: Where "Search" Pays Off (and Where It Rots)

Why bother maintaining the left-&lt;-right invariant? Because it turns search into
**binary search on a tree**. To find a value, compare at the root: smaller means
the answer can only be on the left, larger means only on the right. Every step
throws away half the remaining tree.

```java
TreeNode search(TreeNode root, int target) {
    while (root != null && root.val != target) {
        root = target < root.val ? root.left : root.right;  // discard half
    }
    return root;
}
```

Insertion follows the same descent — walk down as if searching, and hang the new
node where you fall off:

```java
TreeNode insert(TreeNode root, int val) {
    if (root == null) return new TreeNode(val);
    if (val < root.val) root.left  = insert(root.left,  val);
    else                root.right = insert(root.right, val);
    return root;
}
```

### Where It Breaks Down

Both operations are O(height). In a nicely bushy tree, height is **O(log N)** and
life is good. But height is not guaranteed. Insert already-sorted data —
`1, 2, 3, 4, 5` — and each value is larger than the last, so it always goes right:

```
1
 \
  2
   \
    3
     \
      4
```

You've built a **linked list wearing a tree costume.** Height is now N, and
search degrades to **O(N)**. The BST's whole advantage evaporates precisely on the
input you'd most expect (sorted).

This is *why* self-balancing trees exist. **Red-Black trees** and **AVL trees**
rearrange themselves on insert/delete to keep height at O(log N) no matter the
input order. You almost never implement one by hand in an interview — but "why
does a BST degrade, and what fixes it?" is a fair question, and the answer is
exactly this picture. In Java you get a red-black tree for free as `TreeMap` /
`TreeSet` — which is why *those* guarantee O(log N) while a hand-rolled BST does
not.

> A BST is only as good as it is balanced. Sorted input is its worst enemy.
> Balanced variants (Red-Black, AVL) are the fix, and `TreeMap`/`TreeSet` are how
> you get them without writing one.

---

## The Interview Playbook

In an interview, don't think of a tree as a scary data structure. Think of it as
a **graph with no cycles**, which means: no "visited" set needed, and recursion
is safe. Roughly **90% of binary-tree questions are DFS or BFS with extra logic
bolted on.** They cluster into a handful of patterns.

**Pattern 1 — Search / Count.** Does a value exist? How many nodes? Max/min?
→ *Search in a BST*, *Count Complete Tree Nodes*.

**Pattern 2 — Root-to-Leaf Paths.** Carry state *down* the tree (preorder), act
at leaves. → *Path Sum*, *Path Sum II*, *Binary Tree Maximum Path Sum*.

**Pattern 3 — Build / Modify.** Return the (sub)tree you built. → *Invert Binary
Tree*, *Merge Two Binary Trees*, *Construct Binary Tree from Preorder and Inorder*.

**Pattern 4 — Compare Two Trees ⭐** (very common). Recurse both trees in
lockstep. → *Same Tree*, *Subtree of Another Tree*.

**Pattern 5 — Lowest Common Ancestor (LCA).** → *LCA of a Binary Tree*, *LCA of a
BST*.

**Pattern 6 — Validation.** Verify a property holds everywhere. → *Validate BST*,
*Balanced Binary Tree*, *Symmetric Tree*.

Two worked examples show how thin the logic on top of the skeleton really is.

**Invert a tree** — swap every node's children (Pattern 3):

```java
TreeNode invert(TreeNode node) {
    if (node == null) return null;
    TreeNode tmp = invert(node.left);   // build inverted left
    node.left  = invert(node.right);    // ...becomes the right, inverted
    node.right = tmp;
    return node;
}
```

**LCA in a plain binary tree** — the deepest node that has both targets somewhere
below it (Pattern 5):

```java
TreeNode lca(TreeNode node, TreeNode p, TreeNode q) {
    if (node == null || node == p || node == q) return node;
    TreeNode left  = lca(node.left,  p, q);
    TreeNode right = lca(node.right, p, q);
    if (left != null && right != null) return node;  // p and q split here
    return left != null ? left : right;              // both on one side (or neither)
}
```

Both are the DFS skeleton with three or four lines of problem-specific logic.
That's the whole game.

---

## Trees in the Real World

The interview stuff is the tip of the iceberg. The same idea, tuned for
different constraints, runs your whole backend stack:

| Structure | The tweak | Where you've already used it |
|-----------|-----------|------------------------------|
| **B-tree / B+tree** | many children per node, disk-page-sized | Every MySQL/InnoDB index is a B+tree |
| **Red-Black tree** | self-balancing BST | Java's `TreeMap` / `TreeSet` |
| **Trie** | one edge per character | Autocomplete, IP routing, Elasticsearch term dictionaries |
| **Binary heap** | shape + parent/child ordering, stored in an array | `PriorityQueue`, schedulers, top-K |
| **Merkle tree** | nodes hold hashes of their children | Git objects, distributed-system integrity |
| **LSM-tree** | sorted runs merged over time | Write path of many NoSQL engines |

One worth internalizing because it powers your heap grind: a **binary heap** isn't
even stored with pointers — it's packed into a flat array, where a node at index
`i` finds its children by arithmetic:

```
left child  of i  =  2i + 1
right child of i  =  2i + 2
parent      of i  = (i - 1) / 2
```

Same tree idea, zero pointer-chasing. See the heap section in
[data-structures.md](data-structures.md) for the full story.

---

## The 18-Problem Checklist

Solve these and *understand the pattern behind each* — don't memorize solutions —
and you're covered for most binary-tree interviews, especially backend Java roles.

**Fundamentals (get these automatic first):**

- Binary Tree Preorder Traversal
- Binary Tree Inorder Traversal
- Binary Tree Postorder Traversal
- Binary Tree Level Order Traversal
- Maximum Depth of Binary Tree
- Minimum Depth of Binary Tree

**The patterns:**

- Same Tree
- Subtree of Another Tree ⭐
- Symmetric Tree
- Invert Binary Tree
- Diameter of Binary Tree
- Balanced Binary Tree
- Path Sum
- Lowest Common Ancestor of a Binary Tree
- Validate Binary Search Tree
- Kth Smallest Element in a BST
- Binary Tree Right Side View
- Binary Tree Zigzag Level Order Traversal

> The meta-lesson: there are ~6 patterns and 2 skeletons (DFS, BFS). Every problem
> above is one skeleton plus a few lines. Learn the skeletons cold; the rest is
> variation.
