---
title: Recursion — Thinking in Terms of Itself
topic: dsa
---


> Not a reference. A story.
> Recursion feels like magic until you see the one physical thing holding it up:
> a stack of unfinished work. Once you stop trying to trace every call in your
> head and start *trusting* the smaller call, the whole family of "hard" problems —
> trees, permutations, divide-and-conquer, the on-ramp to dynamic programming —
> collapses into the same five-line shape. Read this to build that trust.

See also [data-structures.md](data-structures.md) for the "memory is numbered
boxes" foundation the call stack sits on, [binary-trees.md](binary-trees.md) for
where recursion does its most famous work, and [patterns.md](patterns.md) for the
DFS/backtracking rows in context.

---

## What Recursion Actually Is

A recursive function is one **defined in terms of itself** — it solves a problem
by calling itself on a *smaller* version of the same problem.

That sounds circular, and it would be, except for one physical detail. Walk back
to the [data structures story](data-structures.md): memory is a giant array of
numbered boxes. When a function is called, the machine reserves a small block of
those boxes — a **stack frame** — to hold that call's parameters and local
variables. Call a function from inside a function, and a *second* frame is pushed
on top of the first. The first is frozen mid-execution, waiting.

Recursion is nothing more than a function that keeps pushing frames of *itself*:

```
factorial(4)   ← frame 1, paused at "4 * factorial(3)"
  factorial(3) ← frame 2, paused at "3 * factorial(2)"
    factorial(2)
      factorial(1)  ← this one doesn't recurse; it returns
```

The magic is just the **call stack** doing what it always does. There is no new
mechanism to learn — only a new way to *use* the one you already have.

---

## The Two Parts Every Recursion Has

Every correct recursion has exactly two ingredients. Miss either and it breaks.

1. **The base case** — the smallest input you can answer *without* recursing. It's
   the floor. It's what stops the descent.
2. **The recursive case** — solve one small piece, then hand the rest to a call on
   a **strictly smaller** input, moving toward the base case.

```java
int factorial(int n) {
    if (n <= 1) return 1;          // base case: the floor
    return n * factorial(n - 1);   // recursive case: shrink toward it
}
```

Forget the base case (or fail to actually shrink), and the frames pile up forever
until the stack runs out of boxes:

```java
int broken(int n) {
    return n * broken(n - 1);      // n never stops decreasing past 0
}
// Exception in thread "main" java.lang.StackOverflowError
```

`StackOverflowError` is almost always one thing: **a base case that's missing,
wrong, or never reached.** That's the first place to look.

---

## The Mental Model: Trust the Recursion

Here is the single idea that turns recursion from confusing to obvious.

**Do not trace the whole stack in your head.** You will run out of working memory
around three levels deep, decide recursion is "too hard," and go write a loop.

Instead, take the **leap of faith**: *assume the recursive call already works.*
When you write `factorial(n - 1)`, believe — without checking — that it correctly
returns the factorial of `n - 1`. Your only job is to answer **one** question:

> Given the correct answer for the smaller input, how do I build the answer for
> *this* input?

For factorial: "if I trust that `factorial(n-1)` is `(n-1)!`, then `n!` is just
`n` times that." Done. You never think about `factorial(n-2)` at all.

This is the same reasoning as mathematical induction: prove the base case, then
prove that *if* it holds for `n-1` it holds for `n`. The recursion is the induction,
executed.

---

## The Call Stack, Made Concrete

Trust gets you *writing* recursion. To *debug* it, you sometimes do need to see the
stack — so watch `factorial(4)` once, carefully. It happens in two phases.

**Winding down** — each call pushes a frozen frame and asks a smaller one:

```
factorial(4) → needs 4 * factorial(3)   [frame pushed, waiting]
factorial(3) → needs 3 * factorial(2)   [frame pushed, waiting]
factorial(2) → needs 2 * factorial(1)   [frame pushed, waiting]
factorial(1) → base case, returns 1     [no push — it answers directly]
```

**Unwinding up** — the base case's answer flows back, and each frozen frame
finishes its multiplication as it pops:

```
factorial(1) returns 1
factorial(2) returns 2 * 1  = 2
factorial(3) returns 3 * 2  = 6
factorial(4) returns 4 * 6  = 24
```

The work you wrote *after* the recursive call (the `n *`) runs during unwinding, in
**reverse** order of the calls. That "reverse on the way back up" is the secret
behind post-order tree traversal, reversing a linked list recursively, and printing
a stack top-to-bottom.

---

## Every Recursion Is a Loop in Disguise

Anything a loop can do, recursion can do, and vice-versa — they're equivalent in
power. A loop repeats by *jumping back*; recursion repeats by *calling forward*.
The difference is where the "state between repetitions" lives:

| | Where state lives | How it repeats | Extra memory |
|---|---|---|---|
| Loop | mutable variables | jump back to top | O(1) |
| Recursion | stack frames | call itself | O(depth) frames |

That last column is the catch. The compiler does **not** turn recursion into a loop
for free — every call is a real frame taking real memory. A loop counting to a
million is fine; a plain recursion `count(1_000_000)` will `StackOverflowError`
long before it finishes. Recursion buys you *expressiveness*, and you pay in stack.

So the rule of thumb: reach for recursion when the **problem's shape is recursive**
(the next sections), and reach for a loop when it's a flat, linear repeat.

---

## Transforming Iterative → Recursive

There's a mechanical recipe. Every loop has four parts, and each maps to a piece of
a recursion:

| Loop part | Becomes |
|---|---|
| loop variable (`i`) | a **parameter** |
| accumulator (`sum`) | a **parameter** or the **return value** |
| loop body | the **recursive case** |
| exit condition | the **base case** |

Sum of an array, both ways:

```java
// Iterative
int sum(int[] a) {
    int total = 0;
    for (int i = 0; i < a.length; i++) total += a[i];
    return total;
}

// Recursive — index becomes a parameter, exit condition becomes the base case
int sum(int[] a, int i) {
    if (i == a.length) return 0;        // base: nothing left to add
    return a[i] + sum(a, i + 1);        // this element + the rest (trusted)
}
```

Reverse a string, same recipe:

```java
String reverse(String s) {
    if (s.isEmpty()) return s;                       // base case
    return reverse(s.substring(1)) + s.charAt(0);    // rest reversed, then head last
}
```

Notice you never wrote a loop counter or a mutable accumulator — the stack holds the
in-progress state for you.

---

## Transforming Recursive → Iterative

Going the other way, you replace the *implicit* call stack with an **explicit** one —
an `ArrayDeque` used as a stack (see the "ArrayDeque as Stack" row in
[patterns.md](patterns.md)). You're just managing the frames by hand:

```java
// Recursive DFS
void dfs(Node node) {
    if (node == null) return;
    visit(node);
    for (Node child : node.children) dfs(child);
}

// Iterative DFS — the ArrayDeque *is* the call stack
void dfs(Node root) {
    Deque<Node> stack = new ArrayDeque<>();
    stack.push(root);
    while (!stack.isEmpty()) {
        Node node = stack.pop();
        if (node == null) continue;
        visit(node);
        for (Node child : node.children) stack.push(child);
    }
}
```

**When it's worth doing:** hot paths where frame overhead matters, or recursion deep
enough to risk `StackOverflowError` (a linked list of a million nodes, a graph with a
long path). Otherwise the recursive version is almost always clearer — prefer it.

---

## Head vs. Tail Recursion

*Where* the recursive call sits relative to the rest of the work matters:

- **Head / body recursion** — work happens *after* the call returns, during
  unwinding. `return n * factorial(n - 1)` — the multiply waits for the call.
- **Tail recursion** — the recursive call is the *last* thing the function does;
  there's nothing left to do on the way back up.

```java
// Tail-recursive: the call is the final act, state carried in `acc`
int factorial(int n, int acc) {
    if (n <= 1) return acc;
    return factorial(n - 1, n * acc);   // nothing waits for this
}
```

In languages with **tail-call optimization** (Scala, Kotlin's `tailrec`, most
functional languages), the compiler reuses one frame for tail recursion, making it
as cheap as a loop. **Java does not do this** — the JVM keeps full stack traces for
debugging, so a tail-recursive Java method still pushes a frame per call and can
still overflow. In Java, tail recursion is a *style*, not a free optimization; if you
need the loop's memory profile, write the loop.

---

## When Recursion Is *Easier* (Reach For It)

Recursion shines when the **problem's own definition is recursive** — when "a big X
is made of smaller Xs." Signs you should reach for it:

- **Hierarchical / nested data** — trees, file systems, JSON, HTML. A folder
  contains folders; the code that walks it should contain itself.
- **"Try every combination"** — subsets, permutations, paths, board states. Each
  choice branches into more choices — that's [backtracking](#technique-2--backtracking).
- **Divide and conquer** — the answer for the whole is built from the answer for
  halves (sorting, searching, many geometry problems).
- **Self-similar definitions** — Fibonacci, Ackermann, grammar/parsing rules,
  fractals. The math is already recursive; the code just transcribes it.

---

## When Recursion Is *Worse* (Avoid It)

- **Flat, linear repetition** — summing a list, counting to N. A loop is clearer and
  uses O(1) stack. Don't recurse to look clever.
- **Very deep linear recursion** — depth in the tens/hundreds of thousands risks
  `StackOverflowError`. Either convert to a loop or an explicit stack.
- **Overlapping subproblems, done naively** — the classic trap. Naive `fib` recomputes
  the same values an exponential number of times (next section). Recursion isn't wrong
  here — *un-memoized* recursion is. The fix is caching, which is the door to DP.

---

## Technique 1 — Tree / Graph DFS

This is recursion's home turf. Because a tree is defined recursively (a node plus
subtrees), traversing it is "the same three lines" that [binary-trees.md](binary-trees.md)
keeps pointing at:

```java
void dfs(TreeNode node) {
    if (node == null) return;   // base case: fell off the tree
    dfs(node.left);             // trust: fully handle the left subtree
    dfs(node.right);            // trust: fully handle the right subtree
    // do work here → pre/in/post-order is just *where* you put the visit
}
```

Where you place the "visit this node" line relative to the two calls gives you pre-,
in-, and post-order — the "reverse on unwinding" idea from earlier. Graphs are the
same, plus a `visited` set so cycles don't loop forever (that set replaces the tree's
"no cycles" guarantee).

---

## Technique 2 — Backtracking

Backtracking is recursion for **"explore every possibility"** problems. The shape is
always: **choose → recurse → un-choose.** You make a choice, let the recursion explore
everything that follows from it, then undo the choice and try the next one.

```java
// All subsets of nums
void subsets(int[] nums, int i, List<Integer> path, List<List<Integer>> out) {
    if (i == nums.length) {          // base case: a complete decision made
        out.add(new ArrayList<>(path));
        return;
    }
    subsets(nums, i + 1, path, out);         // choice A: skip nums[i]

    path.add(nums[i]);                        // choose nums[i]
    subsets(nums, i + 1, path, out);          // recurse with it
    path.remove(path.size() - 1);             // un-choose (backtrack)
}
```

The `remove` is the whole trick — it rewinds the shared state so the next branch
starts clean. Permutations, combinations, N-Queens, sudoku, word search: all this
skeleton with a different "is this choice legal?" check.

---

## Technique 3 — Divide & Conquer

Split the problem into independent sub-problems, solve each recursively, then
**combine** the results. Merge sort is the canonical example:

```java
int[] mergeSort(int[] a) {
    if (a.length <= 1) return a;                 // base: already sorted
    int mid = a.length / 2;
    int[] left  = mergeSort(Arrays.copyOfRange(a, 0, mid));   // solve each half
    int[] right = mergeSort(Arrays.copyOfRange(a, mid, a.length));
    return merge(left, right);                   // combine the two sorted halves
}
```

The pattern — *split, solve halves, combine* — is why these algorithms are O(N log N):
`log N` levels of splitting, O(N) work to combine at each level. Binary search is the
degenerate case where one half is thrown away instead of combined.

---

## Technique 4 — Memoization → the Bridge to DP

Naive Fibonacci is the textbook cautionary tale. It's a perfectly correct recursion
that's catastrophically slow, because the call tree recomputes the same values over
and over:

```java
int fib(int n) {
    if (n < 2) return n;
    return fib(n - 1) + fib(n - 2);   // fib(n-2) gets computed again and again...
}
// fib(40) makes ~331 million calls. O(2^n).
```

The recursion tree is fat with **duplicate subtrees** — `fib(2)` alone is recomputed
millions of times. The fix is one line of memory: remember each answer the first time
you compute it.

```java
int fib(int n, Integer[] memo) {
    if (n < 2) return n;
    if (memo[n] != null) return memo[n];         // seen it — reuse, don't recompute
    return memo[n] = fib(n - 1, memo) + fib(n - 2, memo);
}
// Now O(n): each fib(k) is computed once.
```

That single cache collapses O(2^n) to O(n). This is **memoization**, and it is the
front door to **dynamic programming**: DP is just "recursion with overlapping
subproblems, plus a cache." Turn the memo into a filled-bottom-up table and you have
the iterative "tabulation" style. (A full DP note is its own topic — this is only the
bridge to it.)

---

## A Checklist for Writing Any Recursion

When you're stuck, run down this list — it's the whole method:

1. **State the contract in one sentence.** "`f(x)` returns the ___ of `x`." If you
   can't say it, you can't write it.
2. **Find the base case(s).** What's the smallest input you can answer outright?
3. **Shrink toward it.** How do you make the input strictly smaller each call?
4. **Take the leap of faith.** Assume the recursive call already returns the correct
   answer for the smaller input. Don't trace deeper.
5. **Combine.** Given that trusted answer, build the answer for the current input.
6. **Check termination.** Does every path actually reach a base case? (If not: hello,
   `StackOverflowError`.)

Get those six right and the code writes itself — because you defined the problem in
terms of itself, which is all recursion ever was.
