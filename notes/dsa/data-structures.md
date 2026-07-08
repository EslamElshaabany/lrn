---
title: Data Structures — The Mental Model
topic: dsa
---


> Not a reference. A story.
> Every structure here exists because someone hit a wall with the previous one.
> Read this to understand *why* things are the way they are — then the API makes sense on its own.

---

## The One Rule That Explains Everything

Before any structure, there is one physical truth about memory:

**Memory is a giant array of numbered boxes.**

When your program runs, the OS gives it a flat, contiguous block of bytes — each byte has an address. Everything your program does is ultimately reading from or writing to those addresses. Every data structure is just a different *strategy* for organizing those boxes.

Two fundamental questions define every structure:

1. **How are the elements arranged in memory?** (contiguous vs. scattered)
2. **What trade-off does that arrangement create?** (fast reads vs. fast writes, or something in between)

Everything else follows from those two questions.

---

## Chapter 1 — The Array: The Baseline

### The Story

The most natural thing to do with a block of memory is use it directly. An array is just a claim: *"I need N contiguous boxes, all the same size, starting at address X."*

Because the boxes are contiguous and the same size, you can compute the address of any element instantly:

```
address of element[i]  =  base_address  +  (i × element_size)
```

No searching. No following pointers. Just arithmetic. This is why array access is O(1) — it's a single multiplication and addition at the hardware level.

```java
int[] arr = new int[5]; // claim 5 × 4-byte boxes, contiguous in memory

arr[2] = 99;            // go to base + (2 × 4), write 99 there — one instruction
int x  = arr[2];        // same address, read — one instruction
```

### What It's Great At

| Operation | Cost | Why |
|-----------|------|-----|
| Read by index | O(1) | arithmetic |
| Write by index | O(1) | arithmetic |
| Iterate all elements | O(N) | sequential memory = cache-friendly |
| Binary search (sorted) | O(log N) | index arithmetic enables jumping |

### Where It Breaks Down

The array has one brutal limitation: **you must know the size upfront, and it can never change.**

If you need 6 elements but only allocated 5, you're out of luck. You'd have to allocate a new array of size 6, copy everything over, and throw the old one away. That's O(N) work every time you outgrow it.

The second problem is **insertion and deletion in the middle**. Want to insert at index 2? You have to shift every element after it one position right first — O(N). The contiguity that makes reads fast makes structural changes expensive.

```java
// This is what Arrays.sort, Arrays.copyOf, Arrays.fill etc. are:
// just convenient wrappers around direct index manipulation.
int[] a = { 3, 1, 4, 1, 5 };
Arrays.sort(a);                         // O(N log N) — sorts in place
int[] b = Arrays.copyOfRange(a, 1, 4);  // O(N) — copies to new array
Arrays.fill(a, 0);                      // O(N) — overwrites in place
```

### The Mental Model

> An array is a **direct map to memory**. The index IS the address (offset). Speed comes from that directness. Inflexibility comes from the same place.

### The Sorting Rules
- `Raw Arrays:` If you are using a raw array (like int[] or String[]), use `Arrays.sort(myArray)`.
- `Collections:` If you are using a Collection (like ArrayList), use `myList.sort(null)` or `Collections.sort(myList)`.
- `myList.sort() vs Collections.sort():` `myList.sort()` is the modern Java 8+ standard. It's preferred because it allows the specific list type (like your ArrayList) to sort itself internally using the fastest possible algorithm for its structure.
- `The "null" Parameter:` Lists require you to provide a Comparator when sorting. If you just want standard, alphabetical/numerical sorting, you pass null. Note: Java couldn't simply add an empty sort() method directly to lists because adding it might have crashed older legacy codebases.
- `Sorting Custom Types:` If you build a custom object (like a Person), you can sort a list of them using a lambda expression: `myList.sort((p1, p2) -> Integer.compare(p1.age, p2.age)).`

---

## Chapter 2 — ArrayList: Teaching the Array to Grow

### The Story

Arrays are fast but rigid. The obvious fix: hide the resizing behind a wrapper.

`ArrayList` holds an internal `int[]`. When you `add()` and it's full, it silently allocates a new array roughly **1.5× bigger**, copies everything over, and continues. From your perspective, it just works.

```
Initial capacity: 10 (Java's default)
After first overflow: ~15
After second: ~22
... and so on
```

The key insight is *amortized analysis*. Yes, an individual `add()` that triggers a resize is O(N). But if you spread that cost across all the O(1) adds that came before it, the average cost per add is still O(1). This is called **amortized O(1)**.

```java
List<Integer> list = new ArrayList<>(); // internal array: capacity 10, size 0

list.add(1);  // size: 1   — O(1)
list.add(2);  // size: 2   — O(1)
// ... fill to 10
list.add(11); // RESIZE: allocate new[15], copy 10 elements, add 11 — O(N) this one time
list.add(12); // size: 12  — O(1) again
```

### The Trade-offs It Inherits

`ArrayList` inherits the array's layout, so it keeps the array's strengths and weaknesses:

| Operation | Cost | Reason |
|-----------|------|--------|
| `get(i)` / `set(i, v)` | O(1) | still index arithmetic |
| `add(v)` to end | O(1) amortized | resize is rare |
| `add(i, v)` in middle | O(N) | must shift all elements right of i |
| `remove(i)` | O(N) | must shift all elements left of i |
| `contains(v)` | O(N) | no structure for lookup — must scan |

### The Critical Gotcha

```java
list.remove(3);                  // removes element at INDEX 3  — O(N)
list.remove(Integer.valueOf(3)); // removes element with VALUE 3 — O(N)
```

These are two different methods. Java resolves `remove(3)` to the `int` overload (index), not the `Object` overload (value). If your list holds `Integer` objects and you want to remove by value, you must box it.

### When to Prefer Array Over ArrayList

- You know the size at compile time and it never changes
- You're working with primitives (int, double) — `ArrayList<Integer>` boxes every value, adding memory overhead and GC pressure
- Maximum cache performance in a tight loop

### The Mental Model

> `ArrayList` is an array that learned to grow by playing a doubling trick. You pay a one-time O(N) cost occasionally so that every other add is O(1). The layout is still contiguous, so all the array speed is preserved.

### Collections and Primitives

- `No Primitives Allowed:` Java Collections (ArrayList, HashMap, etc.) cannot directly store primitive types (int, double, boolean). They exclusively store Objects.
- `Wrapper Classes Required:` To store primitives, you must use their corresponding Object wrappers (e.g., Integer for int, Double for double, Boolean for boolean). Example: `ArrayList<Integer>`.
- `Autoboxing and Unboxing:` Java handles the conversion automatically behind the scenes. You can seamlessly type `myList.add(5);` and Java will silently wrap 5 into an Integer object before adding it to the list.
- `The Hidden Cost:` Autoboxing is very convenient but has a strict performance penalty. Wrapper objects consume more memory (e.g., 16+ bytes instead of 4 bytes for an int), they heavily fragment data in the CPU cache (pointer chasing), and they create more work for the Garbage Collector. For extreme performance algorithms, always prefer raw arrays (int[]).

---

## Chapter 3 — LinkedList: Scattering Nodes Through Memory

### The Story

Arrays and ArrayLists suffer from one physical constraint: elements must be **contiguous**. That's what forces the expensive shifting on insert/delete.

What if we gave up contiguity entirely? Instead of packing elements into a row, each element becomes an independent **node** that lives anywhere in memory. Each node holds its value *plus a pointer* to the next node.

```
[value | next→] → [value | next→] → [value | next→] → null
```

Now insertion at the front is trivially O(1): allocate a new node, point it at the old head, update the head pointer. No shifting. The structure literally does not care where elements live in memory.

```java
LinkedList<String> ll = new LinkedList<>();

ll.addFirst("C"); // [C]
ll.addFirst("B"); // [B → C]
ll.addFirst("A"); // [A → B → C]
// Nothing moved. Just new nodes pointing at old ones.
```

Java's `LinkedList` is **doubly-linked**: each node has a `next` pointer AND a `prev` pointer. This makes both ends O(1).

### What You Gain and What You Lose

| Operation | Cost | Reason |
|-----------|------|--------|
| `addFirst` / `addLast` | O(1) | just update head/tail pointer |
| `removeFirst` / `removeLast` | O(1) | just update head/tail pointer |
| `get(i)` — random access | O(N) | must walk the chain from the start |
| `contains(v)` | O(N) | must walk the whole chain |
| Memory per element | higher | each node stores value + 2 pointers |

The killer: **no random access**. To find element at index 5, Java literally starts at the head and hops 5 times. This makes `LinkedList` a very poor general-purpose list — `ArrayList` beats it for almost everything except head/tail operations.

### The Cache Problem

Arrays are cache-friendly: elements are adjacent in memory, so loading one loads nearby ones into the CPU cache for free. Linked list nodes are scattered randomly — every hop to a new node is potentially a **cache miss**, meaning the CPU must wait for a round trip to RAM. This makes linked lists significantly slower in practice even when the theoretical complexity is the same.

### When LinkedList Actually Makes Sense

- You only ever touch the head or tail (implementing a deque, LRU cache backbone)
- You have an iterator positioned mid-list and need O(1) insertions at that point
- You never need random access

```java
// As a deque — the only case where LinkedList wins
Deque<String> deque = new LinkedList<>();
deque.addFirst("X");
deque.addLast("Y");
deque.removeFirst();
deque.removeLast();
```

Even then, `ArrayDeque` is usually faster (better cache behavior, no pointer overhead).

### The Mental Model

> `LinkedList` trades the contiguity of arrays for flexibility. Insertions become O(1) because you just redirect pointers. But you lose O(1) access entirely — you can only navigate by walking the chain. In practice, cache effects make it slower than its complexity suggests.

---

## Chapter 4 — ArrayDeque: The Best of Both Worlds (for Ends)

### The Story

`LinkedList` gives O(1) at both ends. `ArrayList` gives O(1) at one end (the back). Can we get O(1) at both ends without the pointer overhead of `LinkedList`?

Yes — with a **circular buffer**. `ArrayDeque` uses a fixed-size array internally but treats it as a ring: the start and end pointers can wrap around. Adding to the front decrements the head pointer (wrapping around if needed), adding to the back increments the tail pointer.

```
Internal array:  [_, _, 3, 4, 5, _, _]
                         ↑head    ↑tail

addFirst(2):     [_, 2, 3, 4, 5, _, _]
addLast(6):      [_, 2, 3, 4, 5, 6, _]
```

When it runs out of space, it resizes like `ArrayList` — but without any random-access use case burdening the design.

This gives `ArrayDeque` the best performance profile for Stack and Queue use cases:

```java
Deque<Integer> dq = new ArrayDeque<>();

// As a Stack (LIFO)
dq.push(1);          // addFirst
dq.push(2);
int top = dq.pop();  // removeFirst → 2

// As a Queue (FIFO)
dq.offer(1);           // addLast
dq.offer(2);
int head = dq.poll();  // removeFirst → 1

// As a Deque (both ends)
dq.offerFirst(0);
dq.offerLast(3);
dq.pollFirst();
dq.pollLast();
```

### Why Not `Stack` or `LinkedList` for This?

- `java.util.Stack` extends `Vector`, which is synchronized on every operation — thread-safety overhead you never asked for
- `LinkedList` works but every node allocates an object on the heap — memory overhead and GC pressure
- `ArrayDeque` is a single contiguous array with two integer pointers — leaner, faster, cache-friendly

### The Mental Model

> `ArrayDeque` is an array that was bent into a circle. By maintaining a head and tail pointer that can wrap around, both ends become O(1) without pointer chasing. Use it as your default Stack and Queue.

---

## Chapter 5 — HashMap: Turning Any Key Into an Index

### The Story

Arrays give O(1) access *if you know the index*. But what if your key is a `String`, an object, or anything other than a small integer?

The insight: **turn the key into an integer (a hash), then use that as an array index.**

```
"alice"  →  hashCode()  →  some big integer  →  % array_size  →  index 3
"bob"    →  hashCode()  →  some big integer  →  % array_size  →  index 7
```

Now you can store `"alice" → 25` by putting 25 at index 3. Lookup is the same path: hash the key, get the index, read it. O(1).

```java
Map<String, Integer> map = new HashMap<>();
map.put("alice", 25); // hash("alice") → index → store 25
int age = map.get("alice"); // hash("alice") → same index → read 25
```

### The Collision Problem

Two different keys can hash to the same index. This is called a **collision**. Java's `HashMap` handles this with **chaining**: each array slot holds a linked list (or, in Java 8+, a balanced tree if the chain gets long). When a collision happens, you just append to the chain.

```
index 3: ["alice"→25] → ["charlie"→30] → null
```

As long as collisions are rare, chains stay short and lookup is still effectively O(1). If a pathological input forces everything into one chain, worst case degrades to O(N) — but this almost never happens with a good hash function.

### Load Factor and Resizing

`HashMap` tracks a **load factor** (default 0.75). When the number of entries exceeds 75% of the internal array size, it doubles the array and rehashes everything. This keeps chains short and maintains O(1) average performance.

```java
// This is why you should specify initial capacity if you know the size:
Map<String, Integer> map = new HashMap<>(1000); // avoid resizing for ~750 entries
```

### Key Operations and Their Complexity

| Operation | Average | Worst case (degenerate hash) |
|-----------|---------|------------------------------|
| `put` | O(1) | O(N) |
| `get` | O(1) | O(N) |
| `containsKey` | O(1) | O(N) |
| `remove` | O(1) | O(N) |
| iteration | O(N + capacity) | — |

### Ordering Variants

| Class | Order | When to Use |
|-------|-------|-------------|
| `HashMap` | none (unpredictable) | most cases — fastest |
| `LinkedHashMap` | insertion order | when you need to iterate in the order items were added |
| `TreeMap` | sorted by key | when you need range queries or sorted iteration |

```java
// LinkedHashMap: remembers insertion order
Map<String, Integer> linked = new LinkedHashMap<>();
linked.put("banana", 2);
linked.put("apple", 1);
linked.put("cherry", 3);
// iterates: banana, apple, cherry (insertion order preserved)

// TreeMap: always sorted by key
Map<String, Integer> tree = new TreeMap<>();
tree.put("banana", 2);
tree.put("apple", 1);
tree.put("cherry", 3);
// iterates: apple, banana, cherry (alphabetical)
```

### The Mental Model

> `HashMap` is an array where the index is computed from the key via hashing. It converts "find by key" into "find by index" — giving you O(1) lookup for any key type. Collisions are the price; the load factor keeps them rare.

---

## Chapter 6 — HashSet: HashMap Without the Values

### The Story

`HashSet` is not a new idea. It's literally a `HashMap` with dummy values:

```
HashSet<String>  ≡  HashMap<String, PRESENT>
                    where PRESENT is a static placeholder object
```

Everything about performance, collision handling, and ordering variants is identical. `HashSet` just exposes the simpler "is this element in the collection?" interface.

```java
Set<Integer> set = new HashSet<>();
set.add(5);             // internally: map.put(5, PRESENT)
boolean has = set.contains(5); // internally: map.containsKey(5)
set.remove(5);          // internally: map.remove(5)
```

| Class | Order | When to Use |
|-------|-------|-------------|
| `HashSet` | none | membership tests, dedup — fastest |
| `LinkedHashSet` | insertion order | when iteration order matters |
| `TreeSet` | sorted | when you need `floor`, `ceiling`, sorted iteration |

### The Mental Model

> `HashSet` is a `HashMap` where you only care about the keys, not the values. Same internals, simpler interface.

---

## Chapter 7 — Trees: Bringing Order to Hierarchical Data

### The Story So Far

Arrays and hash maps are great for flat collections. But some data is inherently hierarchical — file systems, expression parsing, organizational charts — and some problems need data that is *simultaneously* sorted *and* fast to insert/delete.

A sorted array lets you binary search in O(log N), but insertion is O(N) (shifting). A linked list inserts in O(1) at known positions, but search is O(N). Can we get O(log N) for *both*?

Yes — with a **Binary Search Tree (BST)**.

### Binary Search Tree — The Core Idea

Each node holds a value and two pointers (left child, right child). The invariant:

```
Everything in the LEFT subtree < node value
Everything in the RIGHT subtree > node value
```

This lets you search exactly like binary search on a sorted array — at each node, you know which half to go to:

```
        5
       / \
      3   8
     / \   \
    1   4   9

Search for 4:
  - Is 4 < 5? Yes → go left to 3
  - Is 4 > 3? Yes → go right to 4
  - Found. Steps taken: O(height)
```

For a balanced tree, height = O(log N). Both search and insert follow the same path → both are O(log N).

### The Balance Problem

A naive BST degenerates if you insert elements in sorted order:

```
Insert 1, 2, 3, 4, 5 in order:
1
 \
  2
   \
    3
     \
      4  ← this is just a linked list. O(N) search.
```

**Self-balancing BSTs** (like Red-Black Trees, used internally by `TreeMap` and `TreeSet`) automatically rebalance after every insert/delete, guaranteeing O(log N) always.

### TreeMap and TreeSet in Java

Java does not expose a raw BST. Instead you get:

- `TreeMap<K, V>` — a Red-Black Tree where nodes are key-value pairs, always sorted by key
- `TreeSet<E>` — a `TreeMap` with dummy values (same relationship as `HashSet` to `HashMap`)

```java
TreeMap<Integer, String> map = new TreeMap<>();
map.put(5, "e"); map.put(2, "b"); map.put(8, "h");

// The operations arrays and HashMaps can't do:
map.firstKey();       // 2  — smallest key
map.lastKey();        // 8  — largest key
map.floorKey(6);      // 5  — largest key ≤ 6
map.ceilingKey(6);    // 8  — smallest key ≥ 6
map.subMap(2, 6);     // entries with keys in [2, 6)
```

These range-query operations are O(log N) — impossible for HashMap, O(N) for a sorted array scan.

### The Mental Model

> Trees are sorted structures that don't need contiguous memory. They achieve O(log N) for search, insert, and delete by encoding the sorted order in the *shape* of the structure rather than the physical layout of memory. Self-balancing variants guarantee O(log N) regardless of input order.

---

## Chapter 8 — Heap / PriorityQueue: The Fastest Way to the Extreme

### The Story

Sometimes you don't need everything sorted. You just always need the **minimum** (or maximum) instantly. A full sort is overkill.

The **Heap** is a tree that enforces a weaker invariant than a BST:

```
Every parent is smaller than (or equal to) its children.  ← Min-Heap
(The root is therefore always the global minimum.)
```

No left-right ordering between siblings. Just the parent-child rule. This is enough to guarantee the minimum is always at the root, retrievable in O(1).

### The Trick: Store a Tree in an Array

Heaps are always **complete binary trees** (filled level by level, left to right). This means you can store them in a flat array with zero pointers, using index arithmetic:

```
For element at index i:
  Left child:  2i + 1
  Right child: 2i + 2
  Parent:      (i - 1) / 2
```

No nodes. No pointers. Just an array with a clever indexing scheme. This makes heaps extremely cache-friendly.

```
Heap as a tree:        Heap as an array:
       1                [1, 3, 2, 7, 5, 6, 4]
      / \               index: 0  1  2  3  4  5  6
     3   2
    / \ / \
   7  5 6  4
```

### Heap Operations

**Poll (remove minimum):** Take the root (minimum). Fill the hole with the last element. Then "sift down" — swap it with its smaller child repeatedly until the heap property is restored.

**Offer (insert):** Put the new element at the end. "Sift up" — swap it with its parent repeatedly until the heap property is restored.

Both operations touch at most O(log N) nodes (the height of the tree).

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.offer(5); minHeap.offer(1); minHeap.offer(3);

minHeap.peek(); // 1 — O(1), does not remove
minHeap.poll(); // 1 — O(log N), removes and reheapifies
```

### Max-Heap

Java's `PriorityQueue` is a min-heap by default. To flip it, reverse the comparator:

```java
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.offer(5); maxHeap.offer(1); maxHeap.offer(3);
maxHeap.poll(); // 5
```

### The K-th Largest Pattern (Why a Min-Heap?)

The classic trick: to find the K largest elements, maintain a min-heap of size K. The minimum of the heap is the K-th largest overall — anything smaller gets evicted.

```java
// Keep a min-heap of the K largest elements seen so far.
// The top of the heap is the K-th largest.
PriorityQueue<Integer> heap = new PriorityQueue<>();
for (int n : nums) {
    heap.offer(n);
    if (heap.size() > k) heap.poll(); // evict smallest — it can't be in top-K
}
int kthLargest = heap.peek();
```

### The Mental Model

> A Heap enforces only one rule: every parent beats its children. That's enough to guarantee O(1) access to the extreme (min or max). Cleverly stored as a flat array, it avoids pointer overhead entirely. Use it when you need repeated "give me the smallest/largest" — not a full sort.

---

## Chapter 9 — The Stack/Queue Duality: Same Structure, Different Discipline

### The Story

Stack and Queue are not data structures — they are **access disciplines** imposed on a sequence.

- **Stack (LIFO):** Last In, First Out. You can only touch the most recently added item.
- **Queue (FIFO):** First In, First Out. Items leave in the order they arrived.

Both can be implemented on top of `ArrayDeque`. The difference is purely which end you add to and remove from:

```java
Deque<Integer> dq = new ArrayDeque<>();

// Stack discipline (LIFO):
dq.push(1);     // addFirst
dq.push(2);     // addFirst
dq.pop();       // removeFirst → 2 (last added)

// Queue discipline (FIFO):
dq.offer(1);    // addLast
dq.offer(2);    // addLast
dq.poll();      // removeFirst → 1 (first added)
```

### Why Does This Matter for Algorithms?

The discipline you choose changes the *shape* of the traversal:

- **Stack → DFS (Depth-First Search):** push a node, immediately go deep into it before processing its neighbors. Explores one branch completely before backtracking.
- **Queue → BFS (Breadth-First Search):** enqueue a node's neighbors, process them level by level. Finds shortest path in unweighted graphs.

```
Graph: A → B, A → C, B → D, C → D

DFS order (Stack): A → B → D → C  (goes deep immediately)
BFS order (Queue): A → B → C → D  (processes level by level)
```

Same graph. Same code structure. One `stack.pop()` vs `queue.poll()` determines the entire traversal shape.

### The Mental Model

> Stack and Queue are contracts about *which end you're allowed to touch*, not fundamentally different storage structures. The contract you choose determines whether your traversal goes deep first (DFS) or wide first (BFS).

---

## The Full Picture — How It All Connects

```
         All memory is a flat array of bytes
                        │
                        ▼
              ┌─────────────────┐
              │   Array (int[]) │  ← direct use of memory
              │  O(1) access    │
              │  fixed size     │
              └────────┬────────┘
                       │ wrap + resize trick
                       ▼
              ┌─────────────────┐
              │   ArrayList     │  ← grows, but same layout
              │  O(1) amortized │
              │  add at end     │
              └────────┬────────┘
                       │ bend into a ring
                       ▼
              ┌─────────────────┐
              │   ArrayDeque    │  ← O(1) both ends
              │  (Stack/Queue)  │
              └────────┬────────┘
                       │
           ┌───────────┴────────────┐
           │                        │
           ▼                        ▼
    ┌─────────────┐        ┌────────────────┐
    │ Stack (LIFO)│        │  Queue (FIFO)  │
    │     DFS     │        │      BFS       │
    └─────────────┘        └────────────────┘

         Separate path: scatter nodes, add pointers
                        │
                        ▼
              ┌─────────────────┐
              │   LinkedList    │  ← O(1) at ends, O(N) access
              │  pointer-based  │
              └────────┬────────┘
                       │ enforce ordering invariant
                       ▼
              ┌─────────────────┐
              │   BST / Tree    │  ← O(log N) search + insert
              │  sorted order   │
              └────────┬────────┘
                       │ weaken invariant, use array storage
                       ▼
              ┌─────────────────┐
              │      Heap       │  ← O(1) min/max, O(log N) insert
              │  (PriorityQueue)│
              └─────────────────┘

         Entirely different path: hash the key
                        │
                        ▼
              ┌─────────────────┐
              │    HashMap      │  ← O(1) lookup by any key
              │    HashSet      │
              └────────┬────────┘
                       │ + preserve order
                       ▼
              ┌─────────────────┐
              │  LinkedHashMap  │  ← insertion order
              │  LinkedHashSet  │
              └────────┬────────┘
                       │ + enforce sorted order
                       ▼
              ┌─────────────────┐
              │    TreeMap      │  ← sorted + range queries O(log N)
              │    TreeSet      │
              └─────────────────┘
```

---

## The Decision Guide

**Start here every time:**

```
Do you need to look things up by a KEY (not an index)?
├── YES → HashMap family
│         ├── Need sorted keys or floor/ceiling?  → TreeMap / TreeSet
│         ├── Need insertion order?               → LinkedHashMap / LinkedHashSet
│         └── Just membership / lookup?           → HashMap / HashSet
│
└── NO → You're working with a sequence

    Do you always know the size upfront?
    ├── YES + primitives → int[] array (fastest, least overhead)
    └── NO → ArrayList (general default)

    Do you need O(1) at BOTH ends?
    └── YES → ArrayDeque
              ├── LIFO (DFS, undo, parentheses)?  → use as Stack
              └── FIFO (BFS, level order)?        → use as Queue

    Do you always need the MIN or MAX?
    └── YES → PriorityQueue (min-heap by default)

    Do you need sorted order + range queries (floor, ceiling, subMap)?
    └── YES → TreeMap / TreeSet
```

---

## What to Avoid and Why

| Avoid | Use Instead | Reason |
|-------|-------------|--------|
| `java.util.Stack` | `ArrayDeque` | `Stack` extends `Vector`, which is synchronized — you pay thread-safety cost for nothing |
| `LinkedList` as a general list | `ArrayList` | Cache misses make `LinkedList` slower despite equal O() — and you lose O(1) access |
| `LinkedList` as a Queue/Stack | `ArrayDeque` | Same cache problem, plus pointer overhead per node |
| `==` to compare Strings | `.equals()` | `==` compares references (memory address), not content |
| `remove(3)` to remove the value 3 from `List<Integer>` | `remove(Integer.valueOf(3))` | `remove(3)` calls the index overload — silent bug |
| Raw types `List list` | `List<Integer> list` | No compile-time type safety; runtime `ClassCastException` |
| Concatenation in a loop `s += x` | `StringBuilder` | Each `+=` allocates a new `String` — O(N²) total work |
| HashMap when sorted order needed | `TreeMap` | `HashMap` iteration order is unpredictable and can change on resize |


---

*Read this once. Then the LeetCode cheat sheet will make complete sense.*
