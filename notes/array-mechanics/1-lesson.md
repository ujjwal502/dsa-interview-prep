# Array Mechanics — taught from scratch

This is the layer underneath patterns: what a JS/TS array actually **is**, and why some of its operations are instant
while others secretly cost you the whole array. You can't reason about a pattern's
complexity if you don't know the complexity of the primitives it's built from.

Same method as the [Big-O lesson](../big-o/1-lesson.md): build it from one line of
reasoning instead of memorizing a table. Here it is: **an array's cost model comes
entirely from where in memory it lives and how it grows.**

---

## 1. What an array actually is

The classical, fixed-size array (think C) is a single contiguous block of memory. If
element 0 lives at address `base`, and each element takes `size` bytes, element `i` lives
at exactly `base + i * size`. No searching, no following pointers — just arithmetic. That's
**why array indexing is O(1)**: `arr[i]` is one multiply, one add, one memory read, always,
no matter how big the array is.

Compare that to a linked list, where reaching element `i` means walking `i` pointers one at
a time — O(n). The contiguous layout is the *entire* reason arrays give you O(1) access and
lists don't. Keep that trade in your pocket; it explains most of what follows.

---

## 2. But JS arrays resize — so what's the catch?

A fixed-size array can't grow. JS arrays obviously do (`arr.push(x)` just works, no
"array full" error) — so under the hood, the engine (V8, in Chrome/Node) allocates a
backing store **bigger than what you're currently using**, with spare capacity at the end.

- `push` writes into that spare room → O(1).
- When the spare room runs out, the engine allocates a **new, larger** backing store
  (commonly ~1.5–2× the size) and copies every existing element into it — an O(n)
  operation.

That copy is expensive, but it happens rarely — each time it happens, the *next* copy is
twice as far away. Spread that occasional O(n) cost over all the O(1) pushes that happen
between resizes, and the **average cost per push converges to O(1)**. That's what
"amortized O(1)" means: not that every single call is cheap, but that the cost *averaged
over a long sequence* is. `pop` is the mirror image — remove from the end, no one else
moves, O(1).

> This is also why you occasionally hear people say "don't call `.length` in a loop
> condition, cache it" — that advice is stale for arrays (`.length` is O(1), just a stored
> counter). It matters for *live* DOM collections, not plain arrays. Good instinct, wrong
> target.

---

## 3. Why `shift` / `unshift` are a different story

`push`/`pop` only ever touch the **last** slot — nothing else in the array needs to move.
`shift` (remove the first element) and `unshift` (insert at the front) are different: the
front is where index `0` lives, and **every remaining element's index has to change** by
one. The engine can't fake this with spare capacity — it has to physically move every
other element one slot over.

```ts
const arr = [1, 2, 3, 4, 5];
arr.shift();     // removes 1 → engine must shift 2,3,4,5 each one slot left. O(n).
arr.unshift(0);  // inserts 0 at front → same shift, in reverse. O(n).
```

There's no amortization trick that saves you here — every single call is O(n), because
the *number of elements that must move* scales with the array size, always. This is the
single most common hidden-complexity trap in interview code:

```ts
// Looks like it builds a list of n items. It's actually O(n²).
const result: number[] = [];
for (let i = 0; i < n; i++) {
  result.unshift(i);   // O(n) work, done n times → O(n²)
}
```

The fix is almost always: `push` in the order you compute things, then `.reverse()` once
at the end (O(n) total) if order matters — or just push and accept the order, since most
of the time you don't actually need front-insertion at all.

---

## 4. `slice` vs `splice` — the one-letter difference that matters

These two are the most confused pair in the language, and the difference is exactly the
mutate/don't-mutate line:

- **`slice(start, end)`** — **non-mutating**. Returns a **new array** containing a copy of
  the elements from `start` up to (not including) `end`. Cost is proportional to how many
  elements you're copying: O(k) where k is the slice length (worst case O(n) for the whole
  array).
- **`splice(start, deleteCount, ...items)`** — **mutating**. Removes `deleteCount` elements
  starting at `start`, and can insert `...items` in their place, **in the original array**.
  Because it changes the array's length at an arbitrary position, everything after the
  splice point has to shift to close (or open) the gap — O(n), same reasoning as
  shift/unshift, just at an arbitrary index instead of always index 0.

One trick for remembering which is which: **"slice copies, splice splices (into the
original)."** If you need the original untouched, reach for `slice`. If you're editing
in place, `splice`.

---

## 5. Three ways to iterate — and why only one or two should be automatic

**`for (let i = 0; i < arr.length; i++)`** — the classic index loop. Use it when you
actually need the index, or want to break/return early, or need to walk two indices at
once (two pointers). Nothing wrong with it — it's just more to type when you don't need
the index.

**`for (const x of arr)`** — walks the array's **iterator protocol** directly and hands
you each *value*. This should be your default when you just need to touch every element
and don't need the index. Clean, and it works on any iterable (arrays, `Set`, `Map`,
strings), not just arrays.

**`for (const key in arr)`** — walks **enumerable property keys**, as strings. On a plain
array this happens to include the indices (`"0"`, `"1"`, …) — but it *also* picks up any
other enumerable property anyone attached to that array object, and the keys are strings,
not numbers (`arr[key]` still works, but `key + 1` gives you `"01"`, not `1`). **Never use
`for...in` on an array.** It exists for plain objects; on arrays it's a footgun with no
upside over `for...of`.

**`.map(fn)`** — transforms every element and returns a **new array of the same length**,
one output per input: `[1,2,3].map(x => x * 2)` → `[2,4,6]`. It does not mutate the
original. Reach for it when you want a same-shape transformation.

Two neighbors worth knowing cold alongside `.map`:
- **`.forEach(fn)`** — same shape of callback, but returns `undefined`. Use it purely for
  side effects (logging, pushing into an outside array) — never chain off its result,
  because there isn't one.
- **`.filter(fn)`** — returns a new array containing only the elements where `fn` returned
  true. Length can shrink; order is preserved.

`.map`/`.filter` are both O(n) — one pass, one unit of work per element (plus O(n) space
for the new array, unlike `for...of` which needs none).

---

## 6. The one question that predicts all of this

Every rule above collapses into a single question: **does this operation have to touch
every element *after* the point I'm working at?**

- Working at the **end** (`push`/`pop`) → nothing after it → O(1).
- Working at the **front** or **middle** (`shift`/`unshift`/`splice`) → everything after
  shifts → O(n).
- **Copying** a range (`slice`) → cost equals the size of what you copy.
- **Scanning** (`.map`/`.filter`/`for...of`/`.includes`/`.indexOf`) → touches every
  element once → O(n) by itself, but O(n²) the moment you put one **inside** a loop over
  the same array (a very common interview-code trap: `arr.includes(x)` inside a
  `for` loop looks innocent and is quietly quadratic).

You don't need to memorize a table of which methods are O(n) — you need this question
reflexively in your head before you write the line.

---

**Now prove it to yourself →** [Array Mechanics — Test Yourself](3-practice.md).
