# Big-O & Complexity — the language of interviews

Every interview answer ends with the same question: *"What's the time and space
complexity?"* You need to answer it instantly and correctly. This note makes that
automatic.

---

## What Big-O actually measures

Big-O describes **how the work grows as the input grows** — not the wall-clock time.
We throw away constants and lower-order terms because we care about the *shape* of the
growth, not the exact count.

- `3n + 5` → **O(n)** (drop the constant 3 and the +5)
- `n² + n` → **O(n²)** (the n² dominates once n is large)
- `2n + 100` and `0.5n` are **both O(n)** — same shape, different slope

Think of it as: *"If I double the input, roughly what happens to the work?"*

| Class | Name | Double the input → work... | Feels like |
|-------|------|----------------------------|-----------|
| O(1) | constant | unchanged | hash lookup, array index |
| O(log n) | logarithmic | +1 step | binary search |
| O(n) | linear | doubles | one pass through the array |
| O(n log n) | linearithmic | a bit more than doubles | good sorting |
| O(n²) | quadratic | ×4 | nested loop over the same array |
| O(2ⁿ) | exponential | squares | trying every subset (naive) |
| O(n!) | factorial | explodes | trying every permutation (naive) |

---

## How to read complexity off your own code

**Rule 1 — a loop over n items is O(n).**
```ts
for (let i = 0; i < n; i++) { /* O(1) work */ }   // O(n)
```

**Rule 2 — nested loops multiply.**
```ts
for (let i = 0; i < n; i++)
  for (let j = 0; j < n; j++) { /* O(1) */ }        // O(n²)
```

**Rule 3 — sequential (non-nested) work adds, then you keep the biggest.**
```ts
for (...) {}      // O(n)
for (...) {}      // O(n)   → O(n) + O(n) = O(2n) = O(n)
```

**Rule 4 — halving the problem each step is O(log n).**
```ts
while (n > 1) { n = Math.floor(n / 2); }           // O(log n)
```

**Rule 5 — recursion: (number of calls) × (work per call).** Draw the call tree.
Merge sort makes O(log n) levels, each doing O(n) work → **O(n log n)**.

---

## The "which is better" ladder

From fastest to slowest, memorize this order:

```
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)
```

If your brute force is O(n²) and the optimal is O(n), the interviewer wants the O(n).
**The single most common optimization is turning an O(n²) nested-loop lookup into an
O(n) pass by using a hash map** — that's literally what Two Sum teaches (see
[Arrays & Hashing](01-arrays-and-hashing.md)).

---

## Space complexity — don't forget it

Space is the *extra* memory you allocate, not counting the input.

- Storing every element in a hash set → **O(n)** space.
- A handful of pointer variables → **O(1)** space.
- Recursion costs stack space: depth `d` → **O(d)** space even if you allocate nothing.
  A tree DFS on a skewed tree is O(n) space; on a balanced tree it's O(log n).

There's almost always a **time↔space trade-off**. Two Sum's O(n²) solution uses O(1)
space; the O(n) solution "buys" speed with O(n) space for the map. Interviewers love
when you *name* that trade-off out loud.

---

## Common input-size → target-complexity hints

The constraints in a problem quietly tell you the intended complexity:

| n up to... | Intended complexity | Meaning |
|-----------|--------------------|---------|
| ~10–12 | O(n!) / O(2ⁿ) | brute force / backtracking is fine |
| ~20 | O(2ⁿ) | subset enumeration, bitmask DP |
| ~500 | O(n³) | triple loop okay |
| ~5,000 | O(n²) | double loop okay |
| ~10⁵–10⁶ | O(n log n) or O(n) | must sort or single-pass |
| ~10⁹ | O(log n) or O(1) | binary search / math |

When you see `n ≤ 10⁵`, an O(n²) solution will time out — that's your cue to find the
linear or n-log-n approach.

---

## Say it like this in the interview

> "The brute force is O(n²) time and O(1) space because of the nested loop. I can
> trade space for time: with a hash map I do it in a single O(n) pass, at the cost of
> O(n) extra space."

That one sentence signals you understand the trade-off, the brute force, and the
optimization — exactly what they're grading.
