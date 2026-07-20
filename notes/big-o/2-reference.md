# Big-O & Complexity

The vocabulary every interview answer is scored in. Goal: read the complexity off any
code instantly, quote the standard structures/algorithms from memory, and name the
time↔space trade-off out loud.

> This note is the **what** (the lookup). New to Big-O, or want the intuition behind these
> rules? Read the lesson first: [Understanding Big-O — taught from scratch](1-lesson.md).

---

## TL;DR (memorize this block)

**Growth ladder — fastest to slowest:**

```
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2ⁿ) < O(n!)
```

**The 3 simplification rules:**
1. Drop constants → `O(2n)` = `O(n)`, `O(n/2)` = `O(n)`.
2. Drop lower-order terms → `O(n² + n)` = `O(n²)`.
3. Log base is irrelevant → `O(log₂ n)` = `O(log₁₀ n)` = `O(log n)` (differ by a constant).

**Different inputs get different letters** → two loops over arrays of size `n` and `m`
is `O(n + m)` (or `O(n·m)` if nested), **not** `O(n²)`.

---

## The 3 notations

| Notation | Name | Bounds | Means |
|----------|------|--------|-------|
| `O(f)` | Big-O | upper | grows **at most** this fast (worst case) |
| `Ω(f)` | Big-Omega | lower | grows **at least** this fast (best case) |
| `Θ(f)` | Big-Theta | tight | upper **and** lower match (exact growth) |

- Interviews almost always mean **worst-case Big-O** unless they say otherwise.
- "Best / average / worst" are separate axes from O/Ω/Θ. Example: quicksort is `Θ(n log n)`
  average but `O(n²)` worst.

---

## Growth rates — what "grows" actually costs

| Class | Double the input → | n=10 | n=100 | n=1,000 | Typical source |
|-------|--------------------|------|-------|---------|----------------|
| `O(1)` | unchanged | 1 | 1 | 1 | hash lookup, array index, math |
| `O(log n)` | +1 step | ~3 | ~7 | ~10 | binary search, balanced-tree op |
| `O(n)` | doubles | 10 | 100 | 1,000 | single pass |
| `O(n log n)` | slightly >2× | ~33 | ~664 | ~9,966 | good sorting, heap-based |
| `O(n²)` | ×4 | 100 | 10,000 | 1,000,000 | nested loop over same input |
| `O(2ⁿ)` | squares | 1,024 | ~10³⁰ | astronomical | every subset (naive) |
| `O(n!)` | explodes | 3.6M | huge | huge | every permutation (naive) |

---

## Reading complexity off code

| Shape | Complexity | Why |
|-------|-----------|-----|
| single loop `i: 0→n` | `O(n)` | n iterations |
| two **separate** loops | `O(n)` | `n + n = 2n` → drop constant |
| nested loops, both `0→n` | `O(n²)` | n × n |
| nested, inner `0→i` (triangle) | `O(n²)` | `n(n-1)/2` → drop constant |
| loop with `i *= 2` (or `/= 2`) | `O(log n)` | index doubles → ~log₂ n steps |
| loop `n`, inner loop `m` (diff input) | `O(n·m)` | independent sizes |
| outer `0→n`, inner does binary search | `O(n log n)` | n × log n |

```ts
// O(n) — one pass
for (let i = 0; i < n; i++) { /* O(1) */ }

// O(n²) — nested over the same input
for (let i = 0; i < n; i++)
  for (let j = 0; j < n; j++) { /* O(1) */ }

// O(log n) — index doubles each step
for (let i = 1; i < n; i *= 2) { /* O(1) */ }

// O(n log n) — n iterations, each O(log n)
for (let i = 0; i < n; i++) binarySearch(arr, x);
```

> **Rule of thumb:** `total = (number of iterations) × (work inside one iteration)`.
> A method call inside a loop is **not** free — substitute its complexity.

---

## Recursion — recursion tree + Master Theorem

**Method 1 — recursion tree:** `total = (number of calls) × (work per call)`. Draw the
tree; sum the work per level × number of levels.

**Method 2 — Master Theorem** for `T(n) = a·T(n/b) + f(n)` (a ≥ 1, b > 1). Compare
`f(n)` against `n^(log_b a)`:

| Case | Condition | Result |
|------|-----------|--------|
| 1 | `f(n)` grows slower than `n^(log_b a)` | `Θ(n^(log_b a))` |
| 2 | `f(n) = Θ(n^(log_b a))` | `Θ(n^(log_b a) · log n)` |
| 3 | `f(n)` grows faster (+ regularity `a·f(n/b) ≤ k·f(n)`, k<1) | `Θ(f(n))` |

**Common recurrences — just memorize these:**

| Recurrence | Result | Example |
|-----------|--------|---------|
| `T(n) = T(n/2) + O(1)` | `O(log n)` | binary search |
| `T(n) = T(n-1) + O(1)` | `O(n)` | linear recursion (sum a list) |
| `T(n) = T(n/2) + O(n)` | `O(n)` | (geometric, top level dominates) |
| `T(n) = 2·T(n/2) + O(1)` | `O(n)` | tree traversal, build-heap |
| `T(n) = 2·T(n/2) + O(n)` | `O(n log n)` | merge sort, quicksort (avg) |
| `T(n) = T(n-1) + O(n)` | `O(n²)` | recursion doing linear work each level |
| `T(n) = 2·T(n-1) + O(1)` | `O(2ⁿ)` | naive Fibonacci, subset generation |
| `T(n) = n·T(n-1)` | `O(n!)` | permutation generation |

---

## Amortized complexity (a FAANG favorite)

Amortized = average cost **per operation across a long sequence**, even if one operation
is occasionally expensive. Not the same as average-case.

| Operation | Amortized | The occasional expensive step |
|-----------|-----------|-------------------------------|
| Dynamic array `push` | `O(1)` | when full, it doubles capacity → one `O(n)` copy, but rare |
| Hash map `set` / `get` | `O(1)` | a resize/rehash is `O(n)`, but rare; worst single op is `O(n)` on collisions |
| Union-Find `find`/`union` | `~O(1)` | inverse-Ackermann `α(n)` with path compression + union by rank |

> Say it precisely: *"push is **amortized** O(1) — most pushes are O(1), and the rare
> doubling copy averages out."*

---

## Space complexity

**Auxiliary space** = extra memory you allocate, **excluding the input**. This is what
interviewers usually mean. (Total space = input + auxiliary.) Output is sometimes
excluded, sometimes counted — clarify if the result itself is size `n`.

| Source | Space |
|--------|-------|
| a few pointer/counter variables | `O(1)` |
| a hash set/map holding all elements | `O(n)` |
| a 2-D DP table `n × m` | `O(n·m)` |
| recursion — costs **stack frames = max depth** | `O(depth)` |

**Recursion depth examples:**

| Recursion | Stack space |
|-----------|-------------|
| DFS on a **balanced** tree | `O(log n)` |
| DFS on a **skewed** tree / linked list | `O(n)` |
| merge sort | `O(n)` (the merge buffer dominates) |
| quicksort | `O(log n)` avg, `O(n)` worst (call stack) |

---

## Reference: data-structure operations

Average / worst. "Access by index" = jump to position `k`; "Search" = find a value.

| Structure | Access | Search | Insert | Delete | Space | Notes |
|-----------|--------|--------|--------|--------|-------|-------|
| Array (static) | O(1) | O(n) | O(n) | O(n) | O(n) | insert/delete shifts elements |
| Dynamic array | O(1) | O(n) | **O(1)*** at end / O(n) middle | O(n) | O(n) | *amortized at end |
| Stack | O(n) | O(n) | O(1) | O(1) | O(n) | push/pop/peek are O(1) |
| Queue | O(n) | O(n) | O(1) | O(1) | O(n) | enqueue/dequeue O(1) |
| Singly linked list | O(n) | O(n) | **O(1)†** | **O(1)†** | O(n) | †only if you already hold the node |
| Doubly linked list | O(n) | O(n) | **O(1)†** | **O(1)†** | O(n) | †finding the node is still O(n) |
| Hash table | — | O(1) / **O(n)** | O(1) / O(n) | O(1) / O(n) | O(n) | no ordered access; worst = collisions |
| Balanced BST (AVL / Red-Black) | O(log n) | O(log n) | O(log n) | O(log n) | O(n) | worst stays O(log n) |
| Binary heap | O(1)‡ | O(n) | O(log n) | O(log n) | O(n) | ‡peek min/max only; build-heap is O(n) |
| Trie | — | O(L) | O(L) | O(L) | O(Σ·N) | L = key length; Σ = alphabet |

Plain **BST (unbalanced)** degrades to `O(n)` for all ops in the worst case (a sorted
insert order makes it a linked list).

---

## Reference: sorting algorithms

| Algorithm | Best | Average | Worst | Space | Stable? |
|-----------|------|---------|-------|-------|---------|
| Quicksort | Ω(n log n) | Θ(n log n) | **O(n²)** | O(log n) | no |
| Merge sort | Ω(n log n) | Θ(n log n) | O(n log n) | **O(n)** | yes |
| Heapsort | Ω(n log n) | Θ(n log n) | O(n log n) | **O(1)** | no |
| Timsort (JS/Python default) | Ω(n) | Θ(n log n) | O(n log n) | O(n) | yes |
| Insertion sort | Ω(n) | Θ(n²) | O(n²) | O(1) | yes |
| Bubble sort | Ω(n) | Θ(n²) | O(n²) | O(1) | yes |
| Selection sort | Ω(n²) | Θ(n²) | O(n²) | O(1) | no |
| Counting sort | Ω(n+k) | Θ(n+k) | O(n+k) | O(k) | yes |
| Radix sort | Ω(nk) | Θ(nk) | O(nk) | O(n+k) | yes |

Comparison sorts can't beat `O(n log n)`. Counting/radix/bucket beat it by **not
comparing** — they exploit a bounded key range `k`.

---

## Reference: common algorithms

| Algorithm | Complexity | Notes |
|-----------|-----------|-------|
| Binary search | `O(log n)` | sorted input required |
| Tree BFS / DFS | `O(n)` | visit every node once |
| Graph BFS / DFS | `O(V + E)` | V vertices, E edges |
| Topological sort | `O(V + E)` | Kahn's algo / DFS |
| Dijkstra (binary heap) | `O((V + E) log V)` | ≈ `O(E log V)`; no negative edges |
| Bellman-Ford | `O(V · E)` | handles negative edges |
| Floyd-Warshall | `O(V³)` | all-pairs shortest path |
| Union-Find | `~O(α(n)) ≈ O(1)` | path compression + union by rank |
| Kruskal's MST | `O(E log E)` | sort edges + union-find |
| Prim's MST (heap) | `O(E log V)` | |
| KMP substring search | `O(n + m)` | text n, pattern m |
| Generate all subsets | `O(2ⁿ · n)` | 2ⁿ subsets, O(n) to copy each |
| Generate all permutations | `O(n! · n)` | n! perms, O(n) to copy each |

---

## Constraints → target complexity

The input bounds quietly tell you the intended solution.

| n up to… | Target | Approach |
|----------|--------|----------|
| ≤ 10–12 | `O(n!)` | brute force / permutations OK |
| ≤ 20 | `O(2ⁿ)` | subset enumeration, bitmask DP |
| ≤ 100 | `O(n³)` | triple loop OK (Floyd-Warshall etc.) |
| ≤ 1,000 | `O(n²)` | double loop OK |
| ≤ 100,000 | `O(n log n)` | must sort or use a heap |
| ≤ 1,000,000 | `O(n)` | single pass, hashing |
| ≥ 10⁹ | `O(log n)` / `O(1)` | binary search or math |

Modern judges do ~10⁸ simple operations/second. See `n ≤ 10⁵` → an `O(n²)` solution
(10¹⁰ ops) times out → that's the cue to find the `O(n log n)` or `O(n)` route.

---

## Common traps (get these right and you're ahead)

| Trap | Reality |
|------|---------|
| "Two nested loops ⇒ O(n²)" | Only if both run `~n` times. If the inner loop's total work is bounded (e.g. sliding window), it can be `O(n)`. |
| "Sorting then one pass ⇒ O(n)" | The sort is `O(n log n)` — that dominates. |
| Using `arr.includes(x)` / `indexOf` in a loop | Each call is `O(n)` → the loop is `O(n²)`. Use a `Set` for `O(1)` lookups. |
| `arr.unshift()` / `arr.shift()` | `O(n)` — every element shifts. `push`/`pop` are `O(1)`. |
| `arr.slice()`, spread `[...arr]`, `str.substring()` | Allocate a copy → `O(k)` time **and** space. |
| String built with `+=` in a loop | Can be `O(n²)` (strings are immutable). Push to an array, `join` once. |
| Recursion "uses no extra space" | The call stack is `O(depth)` space. |
| "O(n) because the map fits n items" (space) | Correct — but don't forget it counts as `O(n)` *space*, not free. |
| Ignoring two different inputs | `n` and `m` are separate: `O(n + m)` or `O(n·m)`, never silently `O(n²)`. |

JS quick-reference: `Map`/`Set` `get/set/has/delete` and object property access are
`O(1)` average; `Array.push/pop` `O(1)`; `Array.shift/unshift/includes/indexOf` `O(n)`;
`Array.sort` `O(n log n)`.

---

## Say it like this in the interview

> "Brute force is `O(n²)` time, `O(1)` space — the nested loop rescans the array. I can
> trade space for time: with a hash map, each lookup is `O(1)`, so it's a single `O(n)`
> pass at the cost of `O(n)` extra space. `push` into the map is amortized `O(1)`."

That one answer covers brute force, the optimization, the trade-off, and amortization —
exactly what the complexity question is grading.
