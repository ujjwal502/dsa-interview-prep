# Array Mechanics

The lookup table for JS/TS array operations: which mutate, which cost O(n) in disguise,
and which iteration form to reach for. Goal: never write an accidentally-quadratic loop
again.

> This note is the **what** (the lookup). New to why these numbers are what they are?
> Read the lesson first: [Array Mechanics — taught from scratch](1-lesson.md).

---

## TL;DR (memorize this block)

- **End of array is free, front/middle is not.** `push`/`pop` → O(1) amortized.
  `shift`/`unshift`/`splice` → O(n).
- **`slice` copies (safe, new array). `splice` mutates (edits in place).**
- **Default iteration: `for...of` for values, `.map`/`.filter` for transforms. Never
  `for...in` on an array.**
- **A scan inside a loop is quadratic.** `arr.includes(x)` / `arr.indexOf(x)` inside a
  `for` loop over the same array → O(n²). Reach for a `Set`/`Map` instead.

---

## Method complexity & mutation

| Method | Mutates original? | Returns | Complexity | Notes |
|--------|--------------------|---------|-----------|-------|
| `push(x)` | yes | new length | **O(1)** amortized | occasional O(n) resize copy, averages out |
| `pop()` | yes | removed element | **O(1)** | only touches the last slot |
| `shift()` | yes | removed element | **O(n)** | every remaining element re-indexes |
| `unshift(x)` | yes | new length | **O(n)** | same reason, in reverse |
| `slice(a, b)` | **no** | new array (copy) | **O(k)**, k = slice length | up to O(n) for the whole array |
| `splice(i, del, ...items)` | yes | array of removed elements | **O(n)** | shifts everything after index `i` |
| `concat(other)` | no | new array | **O(n + m)** | copies both arrays |
| `indexOf(x)` / `includes(x)` | no | index / boolean | **O(n)** | linear scan — O(n²) if called inside a loop |
| `sort(cmp)` | **yes** | the same array, sorted | **O(n log n)** | V8 uses Timsort; sorts *in place* |
| `reverse()` | **yes** | the same array, reversed | **O(n)** | in place |
| `join(sep)` | no | string | **O(n)** | |
| `Array.from(iterable)` | no | new array | **O(n)** | also works on array-likes (`NodeList`, `arguments`) |
| `[...arr]` (spread) | no | new array (copy) | **O(n)** | same cost as `slice()` with no args |
| `Array.isArray(x)` | no | boolean | **O(1)** | the correct way to check "is this an array" |

**Two traps hiding in that table:**
- `sort()` mutates the array **and** returns it — `const sorted = arr.sort()` looks
  non-mutating but `arr` itself is now sorted too. Copy first (`[...arr].sort(...)`) if
  you need the original order preserved.
- `sort()` with **no comparator** sorts elements as strings (`[10, 2, 1].sort()` →
  `[1, 10, 2]`). For numbers, always pass `(a, b) => a - b`.

---

## Iteration forms

| Form | Gives you | Mutates? | When to use |
|------|-----------|----------|-------------|
| `for (let i = 0; i < arr.length; i++)` | index `i` (you index in yourself) | no | need the index, early break/return, two-pointer walks |
| `for (const x of arr)` | each **value** | no | default choice — just touching every element |
| `for (const key in arr)` | enumerable keys, **as strings** | no | **don't use on arrays** — for plain objects only |
| `arr.forEach((x, i) => …)` | value + index, no return value | no (unless callback mutates) | side effects only; never chain off the result |
| `arr.map((x, i) => …)` | new array, same length | no | same-shape transform |
| `arr.filter((x, i) => …)` | new array, subset | no | keep-if-true |
| `arr.reduce((acc, x) => …, init)` | single folded value | no | sums, grouping, building a `Map`/object from an array |
| `arr.entries()` | iterator of `[index, value]` pairs | no | when `for...of` needs the index too: `for (const [i, x] of arr.entries())` |

---

## DSA idioms you'll use constantly

**Swap two elements without a temp variable:**
```ts
[arr[i], arr[j]] = [arr[j], arr[i]];
```

**Build a result in order — push, don't unshift:**
```ts
// Bad: O(n²)
const result: number[] = [];
for (const x of source) result.unshift(process(x));

// Good: O(n)
const result: number[] = [];
for (const x of source) result.push(process(x));
result.reverse(); // only if you actually need it reversed — O(n), done once
```

**Fast membership check — Set, not `.includes` in a loop:**
```ts
// Bad: O(n²) — .includes scans on every iteration
for (const x of a) if (b.includes(x)) matches.push(x);

// Good: O(n) — build the Set once, O(1) lookups after
const setB = new Set(b);
for (const x of a) if (setB.has(x)) matches.push(x);
```

**Copy an array (don't alias it):**
```ts
const copy = [...original];       // or original.slice()
// const copy = original;         // WRONG — same array, mutations bleed through
```

---

## Common traps

| Trap | Reality |
|------|---------|
| "`unshift`/`shift` are just like `push`/`pop`, backwards" | No — front operations are O(n), end operations are O(1) amortized. |
| `arr.sort()` "just returns a sorted copy" | It mutates `arr` in place *and* returns it. Copy first if you need the original. |
| `arr.sort()` on numbers with no comparator | Sorts lexicographically as strings — `[10, 2, 1]` → `[1, 10, 2]`. Always pass `(a,b) => a-b`. |
| `for...in` on an array | Works "by accident" (indices happen to be enumerable string keys) but is slower and can pick up extra properties. Use `for...of`. |
| `.includes()` / `.indexOf()` inside a loop | Each call rescans — O(n) × loop of O(n) = O(n²). Use a `Set`/`Map`. |
| `const copy = original` | Doesn't copy — both names point at the same array. Use `[...original]` or `.slice()`. |
| `.map()` when you don't use the return value | Wastes an O(n) allocation for nothing — use `.forEach()` or a plain loop for side effects only. |
