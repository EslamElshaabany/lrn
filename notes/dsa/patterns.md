---
title: DSA Cheat Sheet — LeetCode Edition
topic: dsa
---


> Organized by frequency of use. Each section covers: when to use it, key operations with complexities, and the LeetCode patterns it unlocks.

---

## 📋 Quick Reference

| # | Structure | Best For | Key Cost |
|---|-----------|----------|----------|
| 1 | `int[]` Array | Fixed-size, raw speed | O(1) access |
| 2 | `ArrayList` | Dynamic sizing | O(1) add, O(N) remove |
| 3 | `String` / `StringBuilder` | Text manipulation | O(N) most ops |
| 4 | `HashMap` | Key→value lookup, frequency count | O(1) avg |
| 5 | `HashSet` | Membership check, dedup | O(1) avg |
| 6 | `ArrayDeque` as Stack | DFS, parentheses, backtracking | O(1) push/pop |
| 7 | `ArrayDeque` as Queue | BFS, level-order traversal | O(1) offer/poll |
| 8 | `PriorityQueue` | Top-K, K-th largest, heap problems | O(log N) offer/poll |
| 9 | `Arrays` / `Collections` | Sorting, binary search, utilities | O(N log N) sort |
| 10 | `TreeMap` | Sorted keys, range queries | O(log N) all ops |
| 11 | `TreeSet` | Sorted unique elements, floor/ceil | O(log N) all ops |
| 12 | `LinkedList` | O(1) head/tail insert-remove | O(N) random access |
| 13 | `ArrayDeque` as Deque | Sliding window, monotonic deque | O(1) both ends |
| 14 | `Collections` utils | Sort, reverse, shuffle, min, max | varies |

---

## 1. 📦 Arrays — Fixed Size, O(1) Access

**Use when:** size is known and fixed, and you need raw speed.
**Avoid when:** you need dynamic resizing → use `ArrayList` instead.

### Initialization

```java
int[]   a    = new int[5];           // [0, 0, 0, 0, 0]  — zero-initialized by default
int[]   b    = { 1, 2, 3, 4, 5 };   // inline literal
int[][] grid = new int[3][4];        // 2D grid: 3 rows × 4 cols — common in matrix problems
```

### Access & Mutation — O(1)

```java
a[0]     = 10;       // write
int val  = b[2];     // read → 3
```

### Iteration

```java
// index-based (when you need the index)
for (int i = 0; i < b.length; i++) System.out.println(b[i]);

// for-each (when you only need the value)
for (int x : b) System.out.println(x);
```

### Sorting & Utilities — O(N log N)

```java
Arrays.sort(b);               // full sort, ascending
Arrays.sort(b, 1, 4);         // sort subarray [index 1 .. 3] only

Arrays.fill(a, -1);           // [-1, -1, -1, -1, -1]
int[] copy  = Arrays.copyOf(b, b.length);      // full copy
int[] slice = Arrays.copyOfRange(b, 1, 4);     // elements at index [1, 2, 3]

// Binary search — array MUST be sorted first
int idx = Arrays.binarySearch(b, 3);           // returns index, or negative if not found
```

### 🔑 Pattern — 2D Grid: 4-Directional Traversal

```java
int rows = grid.length;
int cols = grid[0].length;

int[][] dirs = { {0,1}, {0,-1}, {1,0}, {-1,0} }; // right, left, down, up

int r = 0, c = 0; // starting cell
for (int[] d : dirs) {
    int nr = r + d[0];
    int nc = c + d[1];
    if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
        // valid neighbor at (nr, nc)
    }
}
```

---

## 2. 📋 ArrayList — Dynamic Array

**Use when:** size is unknown at compile time.
**Avoid:** index-based `remove()` inside a loop → O(N) per call. Use `removeIf()` instead.

### Initialization

```java
List<Integer> list = new ArrayList<>();
```

### Core Operations

```java
// ── Add ───────────────────────────────────────────────────
list.add(10);          // append to end    — O(1) amortized
list.add(0, 99);       // insert at index  — O(N) (shifts everything right)

// ── Access & Mutate ───────────────────────────────────────
int val = list.get(0); // read by index    — O(1)
list.set(0, 42);       // write by index   — O(1)

// ── Remove ────────────────────────────────────────────────
list.remove(Integer.valueOf(10)); // remove by VALUE — O(N)  ⚠️ not remove(10)!
list.remove(0);                   // remove by INDEX — O(N)

// ── Search ────────────────────────────────────────────────
boolean has = list.contains(42);  // O(N)
int     idx = list.indexOf(42);   // O(N), returns -1 if not found

// ── Size ──────────────────────────────────────────────────
int     size  = list.size();
boolean empty = list.isEmpty();
```

### Sorting

```java
Collections.sort(list);                      // ascending   — O(N log N)
list.sort(Comparator.reverseOrder());        // descending  — O(N log N)
```

### Convert To / From Array

```java
Integer[]     arr     = list.toArray(new Integer[0]);
List<Integer> fromArr = new ArrayList<>(Arrays.asList(arr));
```

### 🔑 Pattern — Build Result List

```java
List<List<Integer>> result = new ArrayList<>();
result.add(new ArrayList<>(Arrays.asList(1, 2, 3)));
```

---

## 3. 🔤 String & StringBuilder

**Rule:** `String` is immutable — every `+` in a loop creates a new object.
Use `StringBuilder` for any string building, especially inside loops.

### String — Common Operations

```java
String s = "hello world";

// ── Info ──────────────────────────────────────────────────
int    len    = s.length();          // O(1)
char   ch     = s.charAt(1);        // 'e'        — O(1)
String sub    = s.substring(0, 5);  // "hello"    — O(N)
int    idx    = s.indexOf('o');      // 4          — O(N)

// ── Check ─────────────────────────────────────────────────
boolean starts = s.startsWith("he");
boolean ends   = s.endsWith("ld");

// ── Transform ─────────────────────────────────────────────
String upper   = s.toUpperCase();
String trimmed = s.trim();
String[] parts = s.split(" ");       // ["hello", "world"]
char[]   chars = s.toCharArray();    // mutable char array
```

### Comparison — Never Use `==`

```java
String a = "hello";
String b = new String("hello");

a.equals(b);  // ✅ true  — compares content
(a == b);     // ❌ false — compares memory addresses (references)
```

### StringBuilder — Mutable String Building

```java
StringBuilder sb = new StringBuilder();

sb.append("hello");       // "hello"
sb.append(' ');           // "hello "
sb.append("world");       // "hello world"
sb.insert(5, ",");        // "hello, world"
sb.deleteCharAt(5);       // "hello world"
sb.reverse();             // "dlrow olleh"  — in-place

String result = sb.toString();
```

### 🔑 Pattern — Character Frequency Map (a–z)

```java
String s = "hello world";

int[] freq = new int[26]; // index 0 = 'a', index 25 = 'z'
for (char c : s.toCharArray()) {
    if (c != ' ') freq[c - 'a']++;
}
```

---

## 4. 🗂️ HashMap — Key/Value Store, O(1) Average

**Use when:** lookups by key, frequency counting, memoization, Two-Sum.
**Avoid:** assuming any iteration order — `HashMap` is unordered.
Use `LinkedHashMap` if insertion order matters. Use `TreeMap` if sorted key order matters.

### Core Operations

```java
Map<String, Integer> map = new HashMap<>();

// ── Put / Get ─────────────────────────────────────────────
map.put("alice", 25);                       // O(1)
int age  = map.get("alice");                // O(1) — throws NPE if key missing!
int safe = map.getOrDefault("bob", 0);      // O(1) — safe fallback

// ── Check ─────────────────────────────────────────────────
boolean hasKey = map.containsKey("alice");
boolean hasVal = map.containsValue(25);

// ── Update ────────────────────────────────────────────────
map.put("count", map.getOrDefault("count", 0) + 1); // increment pattern
map.merge("count", 1, Integer::sum);                 // same, more concise

// ── Remove ────────────────────────────────────────────────
map.remove("alice");                        // O(1)
```

### Iteration

```java
for (Map.Entry<String, Integer> e : map.entrySet())
    System.out.println(e.getKey() + " → " + e.getValue());

for (String key : map.keySet())  System.out.println(key);
for (int val   : map.values())   System.out.println(val);
```

### 🔑 Pattern — Frequency Count

```java
String s = "banana";
Map<Character, Integer> freq = new HashMap<>();

for (char c : s.toCharArray()) {
    freq.merge(c, 1, Integer::sum); // increment count for each character
}
```

### 🔑 Pattern — Two Sum (index lookup)

```java
int[] nums   = { 2, 7, 11, 15 };
int   target = 9;

Map<Integer, Integer> seen = new HashMap<>(); // value → index

for (int i = 0; i < nums.length; i++) {
    int complement = target - nums[i];
    if (seen.containsKey(complement)) {
        System.out.println(seen.get(complement) + ", " + i); // found pair
    }
    seen.put(nums[i], i);
}
```

---

## 5. 🟢 HashSet — Unique Items, O(1) Average

**Use when:** membership checks, removing duplicates, tracking visited nodes.
**Avoid:** assuming iteration order. Use `LinkedHashSet` for insertion order.

### Core Operations — All O(1)

```java
Set<Integer> set = new HashSet<>();

set.add(10);
set.add(20);
set.add(10);               // ignored — already present

boolean has = set.contains(10); // true
set.remove(20);
```

### Build From Array (Dedup)

```java
Integer[]    arr    = { 1, 2, 2, 3, 3 };
Set<Integer> unique = new HashSet<>(Arrays.asList(arr)); // {1, 2, 3}
```

### 🔑 Pattern — Detect First Duplicate

```java
int[]        nums    = { 1, 2, 3, 2 };
Set<Integer> visited = new HashSet<>();

for (int n : nums) {
    if (!visited.add(n)) {          // add() returns false if already present
        System.out.println("Duplicate: " + n);
        break;
    }
}
```

---

## 6. 📚 Stack (via ArrayDeque) — LIFO

**Use when:** DFS, balanced parentheses, monotonic stack, undo/backtracking.
**Never use `java.util.Stack`** — it's legacy, synchronized, and slow. Use `ArrayDeque`.

### Core Operations — All O(1)

```java
Deque<Integer> stack = new ArrayDeque<>();

stack.push(1);             // push to top
stack.push(2);
int top    = stack.peek(); // view top (2) — does NOT remove
int popped = stack.pop();  // remove top (2)
boolean empty = stack.isEmpty();
```

### 🔑 Pattern — Valid Parentheses

```java
String         s  = "()[]{}";
Deque<Character> st = new ArrayDeque<>();

for (char c : s.toCharArray()) {
    if (c == '(' || c == '[' || c == '{') {
        st.push(c);
    } else {
        if (st.isEmpty()) { /* invalid */ break; }
        char open = st.pop();
        if ((c == ')' && open != '(') ||
            (c == ']' && open != '[') ||
            (c == '}' && open != '{')) { /* mismatch */ break; }
    }
}
boolean valid = st.isEmpty(); // true if all matched
```

---

## 7. 🚌 Queue (via ArrayDeque) — FIFO

**Use when:** BFS, level-order tree traversal, anything "process in arrival order".
**Prefer `ArrayDeque` over `LinkedList`** — better cache performance.

### Core Operations — All O(1)

```java
Queue<Integer> queue = new ArrayDeque<>();

queue.offer(1);             // add to tail — prefer offer() over add()
queue.offer(2);
int front   = queue.peek(); // view head (1) — does NOT remove
int removed = queue.poll(); // remove head (1)
boolean empty = queue.isEmpty();
```

### 🔑 Pattern — BFS Level-Order Template

```java
// Queue<TreeNode> queue = new ArrayDeque<>();
// queue.offer(root);
//
// while (!queue.isEmpty()) {
//     int levelSize = queue.size();           // number of nodes at this level
//
//     for (int i = 0; i < levelSize; i++) {
//         TreeNode node = queue.poll();
//         // process node here
//         if (node.left  != null) queue.offer(node.left);
//         if (node.right != null) queue.offer(node.right);
//     }
//     // all nodes at this level are done
// }
```

---

## 8. ⛰️ PriorityQueue — Min-Heap / Max-Heap

**Use when:** Top-K elements, K-th largest/smallest, merging sorted lists, Dijkstra.
**Default is a Min-Heap** — the smallest element is always at the top.

### Min-Heap (default)

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

minHeap.offer(5);
minHeap.offer(1);
minHeap.offer(3);

int smallest = minHeap.peek(); // 1 — does NOT remove
int removed  = minHeap.poll(); // 1 — removes smallest
```

### Max-Heap

```java
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());

maxHeap.offer(5);
maxHeap.offer(1);

int largest = maxHeap.poll(); // 5
```

### Custom Comparator — Sort by First Element of int[]

```java
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]); // min by a[0]

pq.offer(new int[]{ 3, 10 });
pq.offer(new int[]{ 1, 20 });

int[] top = pq.poll(); // [1, 20]
```

### 🔑 Pattern — K Largest Elements (min-heap trick)

Keep a min-heap of exactly K elements — the top is always the K-th largest.

```java
int[] nums = { 3, 2, 1, 5, 6, 4 };
int   k    = 2;

PriorityQueue<Integer> kLargest = new PriorityQueue<>(); // min-heap of size k

for (int n : nums) {
    kLargest.offer(n);
    if (kLargest.size() > k) kLargest.poll(); // drop the smallest, keep top-k
}

int kthLargest = kLargest.peek(); // answer: 5
```

---

## 9. 🔀 Sorting Tricks

### Primitive Arrays — In-Place O(N log N)

```java
int[] arr = { 5, 3, 1, 4, 2 };
Arrays.sort(arr);                              // [1, 2, 3, 4, 5]
```

### Object Arrays & Lists — With Comparators

```java
Integer[]    boxed = { 5, 3, 1, 4, 2 };
Arrays.sort(boxed, Comparator.reverseOrder()); // [5, 4, 3, 2, 1]

List<Integer> list = new ArrayList<>(Arrays.asList(5, 3, 1, 4, 2));
Collections.sort(list);                        // [1, 2, 3, 4, 5]
```

### Sort 2D Array by a Column

```java
int[][] intervals = { {2,3}, {1,5}, {4,7} };
Arrays.sort(intervals, (a, b) -> a[0] - b[0]); // sort by first col → [[1,5],[2,3],[4,7]]
```

### Binary Search on Sorted Array — O(log N)

```java
int[] arr = { 1, 2, 3, 4, 5 };           // must be sorted first!
int idx = Arrays.binarySearch(arr, 3);    // returns index 2, or negative if not found
```

> **LeetCode tip:** sorting first simplifies greedy, interval, and two-pointer problems.

---

## 10. 🌳 TreeMap — Sorted Key/Value Store, O(log N)

**Use when:** you need a `HashMap` but with keys in sorted order, or range queries.
**Unique extras:** `firstKey`, `lastKey`, `floorKey`, `ceilingKey`, `subMap`.

### Core Operations — All O(log N)

```java
TreeMap<Integer, String> map = new TreeMap<>();
map.put(3, "c");
map.put(1, "a");
map.put(2, "b");

int    first = map.firstKey();       // 1  — smallest key
int    last  = map.lastKey();        // 3  — largest key
int    floor = map.floorKey(2);      // 2  — largest key  <= 2
int    ceil  = map.ceilingKey(2);    // 2  — smallest key >= 2
Map<Integer, String> sub = map.subMap(1, 3); // keys in [1, 3)  → {1=a, 2=b}

// iterates in ascending key order
for (Map.Entry<Integer, String> e : map.entrySet())
    System.out.println(e.getKey() + " → " + e.getValue());
```

---

## 11. 🌲 TreeSet — Sorted Unique Elements, O(log N)

**Use when:** you need a `HashSet` but with elements in sorted order, or need floor/ceiling.
**Unique extras:** `first`, `last`, `floor`, `ceiling`, `headSet`, `tailSet`.

### Core Operations — All O(log N)

```java
TreeSet<Integer> set = new TreeSet<>();
set.add(5); set.add(1); set.add(3); set.add(7);

int first = set.first();              // 1  — smallest
int last  = set.last();               // 7  — largest
int floor = set.floor(4);             // 3  — largest element  <= 4
int ceil  = set.ceiling(4);           // 5  — smallest element >= 4

SortedSet<Integer> head = set.headSet(5); // {1, 3}     — strictly < 5
SortedSet<Integer> tail = set.tailSet(3); // {3, 5, 7}  — >= 3

for (int val : set) System.out.println(val); // ascending order
```

---

## 12. 🔗 LinkedList — O(1) Head/Tail Insert & Remove

**Use when:** you need constant-time insertions/removals at both ends.
**Avoid:** random index access — it's O(N). Use `ArrayDeque` for pure Stack/Queue needs.

### Core Operations

```java
LinkedList<String> ll = new LinkedList<>();

ll.addFirst("B");              // [B]
ll.addFirst("A");              // [A, B]
ll.addLast("C");               // [A, B, C]

String head = ll.peekFirst();  // "A" — does NOT remove
String tail = ll.peekLast();   // "C" — does NOT remove

ll.removeFirst();              // removes "A" → [B, C]
ll.removeLast();               // removes "C" → [B]

// LinkedList also implements Deque
Deque<String> deque = new LinkedList<>();
deque.push("X");               // add to front (stack-style)
deque.offer("Y");              // add to back  (queue-style)
```

---

## 13. 🎛️ Deque (ArrayDeque) — Double-Ended Queue, O(1) Both Ends

**Use when:** sliding window problems, monotonic deque, or as a combined Stack + Queue.
**Best all-around choice** for Stack and Queue in LeetCode — faster than `LinkedList`.

### Core Operations — All O(1)

```java
Deque<Integer> dq = new ArrayDeque<>();

dq.offerFirst(1);              // add to front
dq.offerLast(2);               // add to back
dq.offerFirst(0);              // [0, 1, 2]

int front = dq.peekFirst();    // 0 — does NOT remove
int back  = dq.peekLast();     // 2 — does NOT remove

dq.pollFirst();                // removes 0 → [1, 2]
dq.pollLast();                 // removes 2 → [1]
```

### 🔑 Pattern — Monotonic Deque (Sliding Window Maximum)

Keep a deque of indices where values are **always decreasing** — front is always the max.

```java
int[] nums = { 1, 3, -1, -3, 5, 3, 6, 7 };
int   k    = 3; // window size

Deque<Integer> mono = new ArrayDeque<>(); // stores indices, not values

for (int i = 0; i < nums.length; i++) {
    // 1. remove indices that have fallen outside the window
    while (!mono.isEmpty() && mono.peekFirst() < i - k + 1)
        mono.pollFirst();

    // 2. remove from back: any index with a smaller value is now useless
    while (!mono.isEmpty() && nums[mono.peekLast()] < nums[i])
        mono.pollLast();

    mono.offerLast(i);

    // 3. window is full — front of deque is the index of the maximum
    if (i >= k - 1)
        System.out.println(nums[mono.peekFirst()]); // output: 3, 3, 5, 5, 6, 7
}
```

---

## 14. 🛠️ Collections Utilities — Static Helper Methods

```java
List<Integer> list = new ArrayList<>(Arrays.asList(3, 1, 4, 1, 5, 9));

// ── Order ──────────────────────────────────────────────────
Collections.sort(list);                            // ascending    — O(N log N)
Collections.sort(list, Comparator.reverseOrder()); // descending   — O(N log N)
Collections.reverse(list);                         // reverse in-place — O(N)
Collections.shuffle(list);                         // random order     — O(N)

// ── Stats ──────────────────────────────────────────────────
int max  = Collections.max(list);                  // largest element  — O(N)
int min  = Collections.min(list);                  // smallest element — O(N)
int freq = Collections.frequency(list, 1);         // count of value 1 — O(N)

// ── Mutate ─────────────────────────────────────────────────
Collections.fill(list, 0);                         // overwrite all with 0
Collections.swap(list, 0, 1);                      // swap elements at index 0 and 1

// ── Safety ─────────────────────────────────────────────────
List<Integer> readOnly = Collections.unmodifiableList(list); // throws on any mutation attempt
```

---

## 🗺️ Roadmap — Coming Next

```
TODO: Trees       — TreeNode, DFS (pre/in/post-order), BFS, LCA
TODO: Graphs      — adjacency list, BFS/DFS template, cycle detection
TODO: DP          — memoization (top-down), tabulation (bottom-up)
TODO: Trie        — prefix tree insert / search
TODO: Binary Search on answer — min/max feasible value template
TODO: Union-Find  — DSU with path compression + rank
TODO: Intervals   — merge, insert, sweep line
```

---

*This cheat sheet is a living document — add more sections as you learn!*
