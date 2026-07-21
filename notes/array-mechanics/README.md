# Array Mechanics

A Week 0 foundation chapter — not a pattern chapter like Week 1's Arrays & Hashing.
Before patterns (two pointers, hashmaps) sit on top of arrays, the raw primitives need to
be automatic: which operations are free, which secretly cost O(n), and how to iterate
without thinking. Work through it in order:

1. **[Lesson](1-lesson.md)** — built from one idea: an array's cost model comes entirely
   from where in memory it lives and how it grows. Derive push/pop/shift/unshift/slice/
   splice's complexity from that, instead of memorizing it.
2. **[Reference](2-reference.md)** — the lookup table: mutation + complexity per method,
   the three iteration forms, and the DSA idioms (safe swap, push-not-unshift, Set over
   `.includes`) you'll reuse everywhere.
3. **[Practice](3-practice.md)** — 12 snippets, answers hidden. Prove you own it.

**Learn → reference → retrieve.** Same loop as [Big-O](../big-o/).
