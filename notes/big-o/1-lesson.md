# Understanding Big-O — taught from scratch

The [reference note](2-reference.md) is the lookup table. This is the
**lesson** — read it start to finish once, and the rules in that table stop being
things to memorize and become things you can *derive*.

We'll build the whole idea in one line of reasoning: **we want to compare algorithms
without running them, so we measure how their cost grows as the input grows.** Everything
else follows from that sentence.

---

## 1. The problem Big-O exists to solve

You've written two functions that do the same job. Which one is "better"?

The obvious idea — run both and time them with a stopwatch — turns out to be useless for
comparing *algorithms*:

- A faster laptop makes both look good. Whose laptop do we standardize on?
- C is faster than Python. Are we comparing the algorithm or the language?
- One input might be tiny, another huge. Which input do we test on?
- A warm cache, background apps, the OS scheduler — all add noise.

Timing measures *this run, on this machine, right now*. We want something that describes
the **algorithm itself**, independent of all that. So we throw away the stopwatch and ask
a different question.

---

## 2. Instead of timing, count the steps

Don't measure seconds — **count the number of basic operations** the algorithm performs,
written as a function of the input size `n`. A "basic operation" is anything that takes
roughly constant time: an addition, a comparison, an array access, an assignment.

Take summing an array:

```ts
function sumArray(nums: number[]): number {
  let total = 0;                 // 1 operation
  for (let i = 0; i < n; i++) {  // the loop body runs n times
    total += nums[i];            // 1 operation, n times → n
  }
  return total;                  // 1 operation
}
```

Count them up: `1 + n + 1 = n + 2` operations. Now checking every *pair* of elements:

```ts
for (let i = 0; i < n; i++)
  for (let j = 0; j < n; j++)
    /* 1 operation */            // runs n × n times → n²
```

That's `n²` operations. We've turned each algorithm into a **cost function**: `n + 2` for
the first, `n²` for the second. This already ignores the machine and the language —
progress. But `n + 2` is fussy. Watch what we do next.

---

## 3. The one insight that makes it all work

Here is the whole trick. **We only care about how cost behaves when `n` gets large** —
because that's exactly when efficiency starts to matter. On tiny inputs everything is
fast; the difference between two algorithms only bites when the input grows.

So let's compare two algorithms at scale. Say algorithm A costs `100n` and algorithm B
costs `n²`. A has a huge constant (100), B has none. Which is better?

| n | A = 100n | B = n² | winner |
|---|---------|--------|--------|
| 1 | 100 | 1 | B |
| 10 | 1,000 | 100 | B |
| 100 | 10,000 | 10,000 | tie |
| 1,000 | 100,000 | 1,000,000 | **A** |
| 1,000,000 | 100,000,000 | 1,000,000,000,000 | **A**, by a million× |

For small `n`, B's lack of a constant makes it look great. But past the crossover, A wins
and keeps winning — by more and more. **The growth rate beat the constant.** The `100`
bought B a head start, but `n²` grows faster than `n`, so it was always going to lose
eventually.

This gives us two rules, and now they're not arbitrary:

- **Constants don't matter.** `100n` and `n` are both straight lines; which one is steeper
  is a machine-and-language detail, not an algorithm difference. We describe both as
  "grows *linearly*."
- **Lower-order terms don't matter.** In `n² + 100n + 500`, once `n` is large the `n²`
  dwarfs everything else. `n = 1000` → the `n²` is a million, the rest is a hundred
  thousand and shrinking in relative terms. We keep only the dominant term.

We're deliberately describing the **shape of the growth**, not the exact operation count.
`n + 2`, `5n`, `100n + 7` are all just "linear."

---

## 4. Big-O is the notation for that shape

**Big-O captures "grows no faster than."** When we say an algorithm is `O(n)`, we mean:
past some input size, its cost never exceeds some fixed multiple of `n`.

That "fixed multiple, past some point" is the entire formal definition, in plain words:

> `f(n) = O(g(n))` if there's a constant `c` and a starting size `n₀` such that
> `f(n) ≤ c · g(n)` for every `n ≥ n₀`.

The `c` is what lets us absorb constants (pick `c = 100` and `100n ≤ c·n`). The `n₀` is
the "past some point" — it's *why* an `O(n²)` algorithm is allowed to be faster than an
`O(n log n)` one on small inputs; Big-O only promises anything once `n` is big enough.

Two companions complete the vocabulary:

- **`Ω(g)`** — "grows *no slower than*" `g`. A lower bound (best case).
- **`Θ(g)`** — "grows *exactly like*" `g`. Upper and lower bounds match.

One more axis, easy to confuse with the above: **best / average / worst case** is about
*which input* you feed the algorithm, not about the O/Ω/Θ symbols. Quicksort is `Θ(n log n)`
on a typical input but `O(n²)` on its worst input. Interviews mean **worst-case Big-O**
unless they say otherwise — it's the only bound that's a guarantee.

---

## 5. Getting a feel for the growth classes

You should be able to *feel* each class. Analogies make them stick:

| Class | Analogy | Why |
|-------|---------|-----|
| **O(1)** | flipping to a page whose number you already know | one jump, size-independent |
| **O(log n)** | finding a name in a phone book by halving | a million names → ~20 flips (`2²⁰ ≈ 10⁶`) |
| **O(n)** | reading every page once | proportional to length |
| **O(n log n)** | the fastest possible sort | do an `O(n)` pass for each of `log n` halving levels |
| **O(n²)** | everyone in a room shaking everyone's hand | `n` people × `n` partners ≈ `n²` |
| **O(2ⁿ)** | trying every on/off setting of `n` switches | each switch doubles the possibilities |
| **O(n!)** | every possible seating order of `n` guests | `n` choices, then `n−1`, then… |

The killer intuition is **"what happens if I double the input?"** A linear algorithm does
twice the work — fine. A quadratic one does *four times* the work. An exponential one
*squares* — that's the cliff. This single question tells you instantly whether an approach
will survive a bigger input.

That halving analogy for `O(log n)` is worth pausing on, because logarithms feel mysterious
until you see it: every step *throws away a constant fraction* of what's left. Halve a
million, you're at half a million, then a quarter… you hit 1 in about 20 steps, because
`2²⁰ ≈ 1,000,000`. Solve `2ᵏ = n` and `k = log₂ n`. **Anything that discards a fixed
fraction each step is logarithmic** — binary search, walking down a balanced tree. (And
the base of the log is just a constant factor, so we don't even write it — `O(log n)`,
full stop.)

---

## 6. How to analyze your own code

Now the practical recipe. The cost of a piece of code is:

> **(number of times it runs) × (cost of one run).**

Apply it structurally:

- **A loop** over `n` items → `O(n)`.
- **Nested loops** → *multiply*. Two loops each running `n` → `O(n²)`.
- **Sequential loops** (one after another, not nested) → *add*, then keep the biggest:
  `O(n) + O(n) = O(n)`.
- **A loop that cuts the range in half** each step (`i *= 2`, or binary search) → `O(log n)`.
- **A method call inside a loop is not free** — substitute *its* cost. A binary search
  (`log n`) inside a loop over `n` items → `O(n log n)`.

**Recursion** uses the same rule, viewed as a tree of calls: total cost = work-per-level ×
number-of-levels. Merge sort splits the array in half each time (so `log n` levels deep),
and at every level it touches all `n` elements once during the merges — `n × log n` =
`O(n log n)`. Draw the call tree when unsure; count the nodes and the work at each.

---

## 7. Space complexity is the same idea, for memory

Everything above measures *time* (operations). **Space complexity** applies the identical
lens to *memory*: how does the extra memory you allocate grow with `n`?

- A few loop variables → `O(1)`, no matter how big `n` is.
- A hash set holding every element → `O(n)`.
- A 2-D table of size `n × m` → `O(n · m)`.

The one people forget: **recursion costs memory even if it allocates nothing**, because
each pending call sits on the call stack. The space is the *maximum depth* of the
recursion — `O(log n)` for a balanced tree, `O(n)` for a skewed one. That's the source of
the classic **time↔space trade-off**: you can often spend memory to save time (a hash map
turning an `O(n²)` scan into an `O(n)` pass) — and naming that trade-off out loud is
exactly what an interviewer wants to hear.

---

## 8. Where this lens takes you

Notice we never memorized anything — we *reasoned*. And the same reasoning keeps paying off:

- **Read constraints backwards.** If a problem says `n ≤ 10⁵`, an `O(n²)` solution is
  `10¹⁰` operations — too slow. The constraint is quietly telling you the target is
  `O(n log n)` or better. You now guess the intended approach before writing a line.
- **Explain surprising results.** The same "sum the work over the structure" move shows
  why building a heap is `O(n)` and not `O(n log n)` (most nodes barely move), and why *no*
  comparison-based sort can ever beat `O(n log n)` (there are `n!` possible orderings to
  distinguish, and you can rule out at most half per comparison). You don't have to take
  those on faith — they fall out of the same counting.
- **Talk like a senior.** "Brute force is `O(n²)` time, `O(1)` space; I can trade space for
  time with a hash map — one `O(n)` pass, `O(n)` extra memory." That's the whole game.

**The mindset to keep:** don't ask "what's the Big-O of this problem?" as if it's a fact to
recall. Ask "how many operations does this do as a function of `n`, and which term grows
fastest?" Answer that, and the Big-O writes itself — for any code, including code you've
never seen.

---

**Now prove it to yourself →** [Big-O — Test Yourself](3-practice.md): 12 snippets,
answers hidden, built to catch the traps this lesson warned about.
