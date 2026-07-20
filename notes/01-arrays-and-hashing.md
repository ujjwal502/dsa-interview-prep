# Arrays & Hashing

The foundational move the rest of the toolkit builds on: **a hash map turns "search"
from O(n) into O(1).** Internalize this and a large fraction of "easy/medium" problems
collapse into a single pass.

---

## The core data structures

### Array
Contiguous block of memory. You get:
- **Index access** `arr[i]` → **O(1)** (the computer jumps straight to the address)
- **Scan / search for a value** → **O(n)** (you must look at each element)
- **Push/pop at the end** → **O(1)** amortized
- **Insert/remove in the middle** → **O(n)** (everything after shifts)

### Hash Map / Hash Set
A structure that stores keys (and optionally values) and answers *"have I seen this?"*
and *"what's the value for this key?"* in **O(1) average time**.

In TypeScript:
```ts
const map = new Map<string, number>();   // key → value
map.set("a", 1);
map.get("a");        // 1
map.has("a");        // true

const seen = new Set<number>();          // just membership
seen.add(5);
seen.has(5);         // true
```

> **Why O(1)?** A hash function turns the key into an array index. You jump straight to
> the bucket instead of scanning. (Worst case is O(n) on pathological collisions, but
> assume O(1) average in interviews.)

Use a **plain object** `{}` only for string/number keys; use `Map` when keys might be
any type or you need insertion order and a clean `.size`. Prefer `Map`/`Set` — they're
what interviewers expect.

---

## The mental model: "trade memory for time"

Almost every Week-1 problem is the same move:

> Instead of a nested loop that re-scans the array (O(n²)), remember what you've seen
> in a hash map so each lookup is O(1). One pass → O(n).

Keep this sentence in your head. When you catch yourself writing a second `for` loop to
"look back" or "look for a match," stop — a hash map probably removes it.

---

## Pattern 1 — Complement lookup (Two Sum family)

**Signal:** "find two elements that combine to a target."

Brute force: check every pair → O(n²). Better: as you walk the array, for each number
`x` ask *"have I already seen `target - x`?"* If a map holds seen values, that's O(1).

```ts
function twoSum(nums: number[], target: number): number[] {
  const seen = new Map<number, number>(); // value → index
  for (let i = 0; i < nums.length; i++) {
    const need = target - nums[i];
    if (seen.has(need)) return [seen.get(need)!, i];
    seen.set(nums[i], i);
  }
  return [];
}
```
Time **O(n)**, space **O(n)**. Full write-up: [`solutions/week-01-arrays-and-hashing/two-sum.ts`](../solutions/week-01-arrays-and-hashing/two-sum.ts).

---

## Pattern 2 — Frequency counting

**Signal:** "anagram", "how many times", "most frequent", "appears more than…".

Count occurrences in a map, then compare or read the counts.

```ts
function isAnagram(s: string, t: string): boolean {
  if (s.length !== t.length) return false;
  const count = new Map<string, number>();
  for (const c of s) count.set(c, (count.get(c) ?? 0) + 1);
  for (const c of t) {
    const n = count.get(c);
    if (!n) return false;        // missing or already exhausted
    count.set(c, n - 1);
  }
  return true;
}
```
Time **O(n)**, space **O(1)** (at most 26 letters / a bounded alphabet).

For **Group Anagrams**, the trick is a *canonical key*: two words are anagrams iff their
sorted letters match, so `"eat"` and `"tea"` both key to `"aet"`. Group by that key.

---

## Pattern 3 — Prefix / suffix products & sums

**Signal:** "product/sum of everything except self", "range sum", "subarray summing to k".

Precompute running totals so each answer is O(1) instead of re-summing.

**Product of Array Except Self** — answer[i] = (product of everything left of i) ×
(product of everything right of i). Two passes, no division, O(n) time O(1) extra space
(the output array doesn't count).

---

## This week's problems (from The Rep Log)

| # | Problem | Difficulty | Pattern |
|---|---------|-----------|---------|
| 1 | Two Sum | Easy | Complement lookup |
| 2 | Valid Anagram | Easy | Frequency counting |
| 3 | Group Anagrams | Medium | Canonical key (sorted string) |
| 4 | Top K Frequent Elements | Medium | Frequency + bucket sort / heap |
| 5 | Product of Array Except Self | Medium | Prefix/suffix products |
| 6 | Valid Palindrome | Easy | Two pointers (bridge to Week 2) |
| 7 | Two Sum II (Sorted) | Medium | Two pointers on sorted input |
| 8 | 3Sum | Medium | Sort + two pointers |
| 9 | Container With Most Water | Medium | Two pointers, greedy shrink |

Notice the arc: problems 1–5 are hashing; 6–9 quietly introduce **two pointers**, next
week's headline pattern. That overlap is intentional — patterns bleed into each other.

---

## How to practice this week (the Rep Log loop)

1. Attempt **Two Sum** cold with a 25-min timer. Try the O(n²) brute force first if
   nothing else comes — then ask "how do I remove the inner loop?"
2. Once the hash-map idea clicks, close everything and re-implement from an empty file.
3. Move to Valid Anagram, then Group Anagrams. Each reuses the map idea.
4. When you can name "this is a complement-lookup / frequency-count problem" within 2
   minutes of reading it, the week is done.

**Common mistakes to avoid:**
- Adding `nums[i]` to the map *before* checking for its complement (breaks when the
  same index would be reused). Check first, then insert.
- Using `if (seen.get(x))` when the stored value could be `0` — use `seen.has(x)`.
- Forgetting that `map.get()` returns `undefined` for missing keys, not `0`.
