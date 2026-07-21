# DSA Interview Prep

A **pattern-first system for revising data structures & algorithms** for senior and
FAANG-level interviews — built around spaced repetition and fast pattern recognition,
the two things that actually move the needle once you already know how to code.

This repo doubles as a reference: the notes, templates, and pattern cheatsheets are
written to be **forked and followed**, so anyone can run the same loop instead of
grinding random problems. Solutions are in **TypeScript**; notes are language-neutral.

> **The thesis:** interview performance is a retrieval problem, not a knowledge problem.
> You don't fail because you've never seen a sliding window — you fail because you
> didn't *recognize* it in time. This system optimizes for recognition and recall.

---

## The Curriculum (14 weeks, 2 phases)

Phase 1 is the core toolkit; Phase 2 layers on the patterns that separate senior
candidates. The difficulty mix per topic is deliberate — easies build fluency, mediums
are the interview bar, hards prove you can *combine* patterns under pressure.

| # | Topic | The one idea |
|---|-------|--------------|
| 01 | Arrays & Hashing | Trade memory for time with a hash map |
| 02 | Sliding Window · Stack | Grow/shrink a window; remember what's still open |
| 03 | Binary Search · Linked List | Halve the search space; pointer manipulation |
| 04 | Trees (BFS / DFS) | Learn the DFS shape once — most tree problems reuse it |
| 05 | Tries · Heaps · Backtracking | Prefix trees, top-k, try/undo/repeat |
| 06 | Backtracking · Graphs (intro) | Finish backtracking; flood-fill traversal |
| 07 | Graphs · 1-D DP (intro) | Topological sort; the mindset shift into DP |
| 08 | DP Consolidation · Speed Round | Retrieval under a timer |
| 09 | 2-D Dynamic Programming | Two changing inputs → a table |
| 10 | Advanced Graphs · Intervals · Greedy | Dijkstra, Union-Find, sweep, greedy |
| 11 | Curated Hards | The repeat offenders that stack two known patterns |
| 12 | Bit Manipulation | XOR cancels pairs; `n & (n-1)` drops the lowest set bit |
| 13 | Math & Geometry | In-place matrix bookkeeping; overflow-safe math |
| 14 | Design · Adv. Graphs · Intervals · Greedy | LRU Cache, Meeting Rooms II, Alien Dictionary — senior staples |

---

## The System (why this works better than grinding)

Every problem runs through a spaced-repetition loop designed to make solutions *stick*,
not just get solved once:

1. **Attempt cold** — 25–30 min timer, genuine attempt. The struggle is what builds the memory.
2. **Unstick, don't spoil** — Read only enough of a solution to get moving. Never the whole thing.
3. **Re-implement blank** — Once it clicks, close everything and rewrite from an empty file until it passes.
4. **Schedule the redo** — Mark it solved; it re-enters the queue 3–4 days out.
5. **Prove retention** — A clean solve from memory on the redo means it's truly learned.

The non-obvious rule: before writing any code on a redo, **name the pattern in under two
minutes.** Pattern recognition is the actual interview skill — the
[pattern cheatsheet](patterns/pattern-recognition-cheatsheet.md) is the index for it.

---

## Repo Structure

```
dsa-interview-prep/
├── notes/            Concept notes — one folder per chapter (e.g. big-o/)
├── patterns/         "When you see X → reach for Y" recognition cheatsheet
├── templates/        Reusable, memorized TS templates (binary search, DFS, backtracking…)
├── solutions/        One folder per topic; one .ts file per problem, self-checking
└── progress/         A running log of what's been covered
```

Each solution file follows the same shape: problem link, approach in plain English,
complexity analysis, a clean implementation, and inline test assertions.

---

## Running solutions

Solutions are self-checking — assertions at the bottom of each file run on execution.

```bash
npm install          # one-time: installs tsx
npx tsx solutions/week-01-arrays-and-hashing/two-sum.ts
```

Passing assertions print nothing and exit `0`; a failing assertion throws.

---

## Using this repo yourself

Fork it, work chapter by chapter, and run each problem through the five-step loop above.
The notes give you the mental model, the templates give you the muscle memory, and the
cheatsheet trains the recognition. Start with the foundations chapters —
**[Big-O](notes/big-o/)** for the vocabulary everything else is scored in, then
**[Array Mechanics](notes/array-mechanics/)** for the primitives Week 1's patterns are
built on top of.
