# Big-O — Test Yourself

Reading about Big-O doesn't build the skill; *retrieving* it does. Do this like a rep from
your tracker's loop:

1. Read the snippet.
2. **Commit to an answer out loud** — time complexity, space complexity, and *why* — before
   you reveal anything.
3. Open the answer. If you were right for the right reason, move on. If not, that snippet
   goes in your redo pile.

Assume `n` is the input size (array/string length) unless noted. Answers are collapsed —
no peeking.

---

## Tier 1 — warm-up

### Q1
```ts
function endpoints(nums: number[]): number {
  return nums[0] + nums[nums.length - 1];
}
```
<details><summary>Answer</summary>

**Time: O(1)** · **Space: O(1)**

A fixed number of operations no matter how big the array is. Indexing and reading `.length`
are both `O(1)` — you don't touch the other elements.
</details>

### Q2
```ts
function sum(nums: number[]): number {
  let total = 0;
  for (const x of nums) total += x;
  return total;
}
```
<details><summary>Answer</summary>

**Time: O(n)** · **Space: O(1)**

One pass over `n` elements. Only a single accumulator variable is allocated, so space is
constant.
</details>

---

## Tier 2 — reading loops

### Q3
```ts
function minMax(nums: number[]): [number, number] {
  let min = Infinity, max = -Infinity;
  for (const x of nums) if (x < min) min = x;   // n
  for (const x of nums) if (x > max) max = x;    // n
  return [min, max];
}
```
<details><summary>Answer</summary>

**Time: O(n)** · **Space: O(1)**

Two *sequential* loops → `n + n = 2n` → drop the constant → `O(n)`. Sequential work **adds**;
only nesting multiplies. (A single loop tracking both min and max is the same complexity —
just a smaller constant.)
</details>

### Q4
```ts
function anyEqualPair(nums: number[]): boolean {
  for (let i = 0; i < nums.length; i++)
    for (let j = 0; j < nums.length; j++)
      if (i !== j && nums[i] === nums[j]) return true;
  return false;
}
```
<details><summary>Answer</summary>

**Time: O(n²)** · **Space: O(1)**

Nested loops, each running `n` times → `n × n`. (A hash `Set` would cut this to `O(n)` time
for `O(n)` space — the classic trade.)
</details>

### Q5
```ts
function countPairs(nums: number[]): number {
  let count = 0;
  for (let i = 0; i < nums.length; i++)
    for (let j = i + 1; j < nums.length; j++)   // note: j starts at i + 1
      count++;
  return count;
}
```
<details><summary>Answer</summary>

**Time: O(n²)** · **Space: O(1)**

The trap: the inner loop runs fewer times each round — `(n-1) + (n-2) + … + 1 = n(n-1)/2`.
That's `≈ n²/2` → drop the constant → **still `O(n²)`**. Doing "half the work" of a full
nested loop doesn't change the growth class.
</details>

### Q6
```ts
function halvings(n: number): number {
  let steps = 0;
  while (n > 1) { n = Math.floor(n / 2); steps++; }
  return steps;
}
```
<details><summary>Answer</summary>

**Time: O(log n)** · **Space: O(1)**

Each step throws away half the remaining value, so it takes `log₂ n` steps to reach 1
(solve `2ᵏ = n`). Any loop that cuts the range by a constant fraction is logarithmic.
</details>

---

## Tier 3 — the tricky middle

### Q7
```ts
function hasCloseElements(nums: number[]): boolean {
  nums.sort((a, b) => a - b);                       // ?
  for (let i = 1; i < nums.length; i++)             // ?
    if (nums[i] - nums[i - 1] < 1) return true;
  return false;
}
```
<details><summary>Answer</summary>

**Time: O(n log n)** · **Space: O(n)** (or `O(log n)`, engine-dependent)

The sort is `O(n log n)`; the pass is `O(n)`. Sum them and keep the biggest →
`O(n log n)`. **The sort dominates** — this is why "sort, then scan" is almost always
`O(n log n)`, not `O(n)`. The sort itself needs memory (V8's Timsort is up to `O(n)`), so
don't claim `O(1)` space just because *you* didn't allocate anything.
</details>

### Q8
```ts
function intersectionSize(a: number[], b: number[]): number {
  const setA = new Set(a);                    // build over a
  let count = 0;
  for (const x of b) if (setA.has(x)) count++; // scan b
  return count;
}
```
<details><summary>Answer</summary>

**Time: O(n + m)** · **Space: O(n)** (n = `a.length`, m = `b.length`)

Two *different* inputs get two *different* variables — building the set is `O(n)`, scanning
`b` is `O(m)`. It is **not** `O(n²)`; collapsing distinct sizes into one letter is a common
error. (If instead you nested a loop over `b` inside a loop over `a`, *that* would be
`O(n·m)`.)
</details>

### Q9
```ts
function firstUnique(nums: number[]): number {
  for (const x of nums) {
    if (nums.indexOf(x) === nums.lastIndexOf(x)) return x; // look closer
  }
  return -1;
}
```
<details><summary>Answer</summary>

**Time: O(n²)** · **Space: O(1)**

The hidden cost: `indexOf` and `lastIndexOf` each **scan the whole array** — `O(n)` — and
they're inside an `O(n)` loop → `O(n²)`. A method call in a loop is never free; substitute
its cost. **Fix:** count frequencies in a `Map` first (`O(n)` time, `O(n)` space), then find
the first value with count 1.
</details>

### Q10
```ts
function longestUniqueSubstring(s: string): number {
  const seen = new Set<string>();
  let left = 0, best = 0;
  for (let right = 0; right < s.length; right++) {
    while (seen.has(s[right])) {   // a loop inside a loop — so O(n²)?
      seen.delete(s[left]);
      left++;
    }
    seen.add(s[right]);
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```
<details><summary>Answer</summary>

**Time: O(n)** · **Space: O(min(n, alphabet))**

The important one. It *looks* `O(n²)` because of the nested `while`, but `left` only ever
moves **forward** and never resets. Across the entire run it can advance at most `n` times
total. So each character is added once and removed at most once → `~2n` operations → `O(n)`.
This is **amortized/aggregate** reasoning: don't multiply loop bounds blindly — ask how many
times the inner work can *ever* run in total. This is why every sliding-window solution is
`O(n)`.
</details>

---

## Tier 4 — recursion & space

### Q11
```ts
function fib(n: number): number {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}
```
<details><summary>Answer</summary>

**Time: O(2ⁿ)** · **Space: O(n)**

Each call spawns two more, and the tree is ~`n` levels deep → the number of calls grows
exponentially (tight bound is `O(φⁿ) ≈ 1.618ⁿ`, which sits inside `O(2ⁿ)`). But **space is
only `O(n)`**: recursion explores depth-first, so at any instant the call stack is at most
`n` frames deep. Classic case where time and space complexities differ sharply. (Memoizing
collapses time to `O(n)`.)
</details>

### Q12
```ts
// TreeNode = { val: number; left: TreeNode | null; right: TreeNode | null }
function maxDepth(root: TreeNode | null): number {
  if (!root) return 0;
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```
<details><summary>Answer</summary>

**Time: O(n)** · **Space: O(h)** — `O(log n)` balanced, `O(n)` skewed

Every node is visited exactly once → `O(n)` time. Space is the recursion-stack depth, which
equals the tree's **height** `h`: a balanced tree is `O(log n)` deep, a degenerate
(linked-list-shaped) tree is `O(n)` deep. When a bound depends on shape, **state both the
balanced and worst case** — interviewers listen for it.
</details>

---

## What "good" looks like

You're ready when, for any snippet, you can state **time, space, and the one-sentence
reason** in under 30 seconds — and you never fall for Q5 (triangular is still `O(n²)`), Q9
(hidden scan), or Q10 (sliding window is `O(n)`). Those three catch the most people.

Next: apply this live. Every solution file in [`solutions/`](../../solutions/) ends with its
complexity — predict it before you read it.
