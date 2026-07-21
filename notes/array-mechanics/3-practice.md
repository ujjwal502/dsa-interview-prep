# Array Mechanics — Test Yourself

Same loop as always:

1. Read the snippet.
2. **Commit to an answer out loud** — before you reveal anything.
3. Open the answer. Right for the right reason → move on. Otherwise, redo pile.

Answers are collapsed — no peeking.

---

## Tier 1 — name the complexity

### Q1
```ts
const arr = [1, 2, 3, 4, 5];
arr.push(6);
arr.pop();
```
<details><summary>Answer</summary>

**Amortized O(1)** for `push`, plain **O(1)** for `pop`. Both only touch the last slot —
nothing else in the array needs to move. (Only `push` needs the "amortized," for the rare
resize copy; `pop` never copies.)
</details>

### Q2
```ts
const arr = [1, 2, 3, 4, 5];
arr.unshift(0);
arr.shift();
```
<details><summary>Answer</summary>

**O(n)** for both. Every other element has to re-index by one slot — front operations
can't be amortized away like `push`/`pop` can.
</details>

### Q3
```ts
const arr = [1, 2, 3, 4, 5];
const copy = arr.slice(1, 3);
```
<details><summary>Answer</summary>

**O(k)** where k = the slice length (here, 2). Non-mutating — `arr` is untouched, `copy`
is a brand-new array.
</details>

### Q4
```ts
const arr = [1, 2, 3, 4, 5];
arr.splice(2, 1, 10, 11);
```
<details><summary>Answer</summary>

**O(n)**. It mutates `arr` in place, and everything after index 2 has to shift to make
room for the extra inserted element (removing 1, inserting 2 → net length +1).
</details>

---

## Tier 2 — mutates or not?

### Q5
```ts
const original = [3, 1, 2];
const sorted = original.sort((a, b) => a - b);
console.log(original);
```
<details><summary>Answer</summary>

Prints **`[1, 2, 3]`** — `original` itself is now sorted. `sort()` mutates in place and
also returns the same array; `sorted` and `original` are the same reference. If you
needed `original` untouched, you'd need `[...original].sort(...)`.
</details>

### Q6
```ts
const original = [3, 1, 2];
const sliced = original.slice();
console.log(original === sliced);
```
<details><summary>Answer</summary>

Prints **`false`**. `slice()` with no arguments still returns a **new** array (a shallow
copy of the whole thing) — same values, different reference. This is the standard
"copy an array" idiom, along with `[...original]`.
</details>

### Q7
```ts
const a = [1, 2, 3];
const b = a;
b.push(4);
console.log(a);
```
<details><summary>Answer</summary>

Prints **`[1, 2, 3, 4]`**. `const b = a` does not copy — `b` and `a` are two names for
the *same* array, so mutating through `b` is visible through `a`. This is the "aliasing"
trap; use `[...a]` if you wanted an independent copy.
</details>

---

## Tier 3 — spot the trap

### Q8
```ts
function buildReversed(nums: number[]): number[] {
  const result: number[] = [];
  for (const x of nums) {
    result.unshift(x);
  }
  return result;
}
```
<details><summary>Answer</summary>

**O(n²)**, not O(n). It *looks* like a simple O(n) build, but `unshift` is O(n) and it
runs n times → O(n²). Fix: `result.push(x)` in a loop that walks `nums` backwards, or
push in forward order and `.reverse()` once at the end (O(n) total).
</details>

### Q9
```ts
function hasCommonElement(a: number[], b: number[]): boolean {
  for (const x of a) {
    if (b.includes(x)) return true;
  }
  return false;
}
```
<details><summary>Answer</summary>

**O(n · m)** (n = `a.length`, m = `b.length`) — worst case O(n²) if similar sizes.
`b.includes(x)` rescans all of `b` on every iteration of the outer loop. Fix: build
`const setB = new Set(b)` once (O(m)), then `setB.has(x)` is O(1) per check → overall
O(n + m).
</details>

### Q10
```ts
let total = 0;
const arr = [10, 20, 30];
for (const key in arr) {
  total += key;
}
console.log(total);
```
<details><summary>Answer</summary>

Prints **`"0012"`**, not `3` or `60`. Two stacked traps: `for...in` yields keys as
**strings** (`"0"`, `"1"`, `"2"`), not the values or numeric indices — and `total += key`
does string concatenation, not addition, once `total` stops being a number. Step by step:
`0 + "0"` → `"00"`, then `+ "1"` → `"001"`, then `+ "2"` → `"0012"`. This is
exactly why `for...in` doesn't belong on arrays: use `for (const x of arr)` if you want
values, or `arr.entries()` if you need the index too.
</details>

---

## Tier 4 — apply it

### Q11
Rewrite this so it's O(n) instead of O(n²), without changing what it returns:
```ts
function dedupe(nums: number[]): number[] {
  const result: number[] = [];
  for (const x of nums) {
    if (!result.includes(x)) result.push(x);
  }
  return result;
}
```
<details><summary>Answer</summary>

```ts
function dedupe(nums: number[]): number[] {
  const seen = new Set<number>();
  const result: number[] = [];
  for (const x of nums) {
    if (!seen.has(x)) {
      seen.add(x);
      result.push(x);
    }
  }
  return result;
}
```
The original's `result.includes(x)` rescans the (growing) result array on every
element — O(n²). Tracking membership in a `Set` alongside `result` makes every check
O(1), for an overall O(n).
</details>

### Q12
What's the complexity of this in-place reversal, and why doesn't it need a temp array?
```ts
function reverseInPlace(arr: number[]): void {
  let left = 0, right = arr.length - 1;
  while (left < right) {
    [arr[left], arr[right]] = [arr[right], arr[left]];
    left++;
    right--;
  }
}
```
<details><summary>Answer</summary>

**Time: O(n) · Space: O(1)**. Two pointers walk toward each other, swapping as they go —
`n/2` swaps total, still O(n) after dropping the constant. No temp *array* is needed
because the swap only ever holds two elements at a time (via destructuring), not a copy
of the whole array — that's the O(1) space, versus `[...arr].reverse()` which is O(n)
space for the copy.
</details>

---

## What "good" looks like

You're ready when `push`/`pop` vs `shift`/`unshift` is instant recall (not something you
derive), you never write `.includes()`/`.indexOf()` inside a loop over the same data
without reaching for a `Set` instead, and `for...of` is your reflex — `for...in` on an
array should look wrong to your eye on sight.

Next foundation item: **Map & Set fluency** — the data structure that makes the `Set`
fixes above O(1) in the first place.
