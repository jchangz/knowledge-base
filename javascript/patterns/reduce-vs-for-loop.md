---
tags: [array, iteration]
date: 2026-07-28
type: pattern
---

# Reduce vs for-loop (when mutating an accumulator)

**TL;DR:** Just use a `for...of` loop.

- `reduce()` implies each step hands back a brand new value — "We are transforming state cleanly and immutably."
  - if you mutate the accumulator, it's the same object the whole time, so that promise never really gets kept.
- A loop never made that promise in the first place, so it's just as fast (O(n)) without the mixed signal.
- The only time it's worth making `reduce` actually build a new value each step (O(n²)) is when something downstream genuinely needs that new reference or a history of every step — React/Redux state, undo, time-travel debugging. Otherwise, `reduce` isn't buying you anything a loop doesn't already do.

**Scope:** everything above is about accumulating into a *compound* value — an object or array that has to be copied to produce something new. It doesn't apply to scalar accumulators:

```js
// O(n), no copy penalty, nothing to mutate in the first place —
// numbers are primitives, so a new one is produced "for free" every step.
const sum = [1, 2, 3, 4].reduce((acc, num) => acc + num, 0)
```

A primitive (number, string, boolean) can't be mutated in place at all, so there's no O(n²) risk and no honesty-vs-cost tradeoff to weigh — `reduce` is simply the right, cheap tool for summing/counting/max-min. The for-loop-by-default advice is specifically about the object/array case above, not reduce in general.

```
DECLARATIVE (reduce + spread) -- a NEW object every step

+----+     +-----------+     +-----------------+             +--------------------------------+
| {} | --> | {Alice:1} | --> | {Alice:1,Bob:1} | --> ... --> | {Alice:2,Bob:1,Tiff:1,Bruce:1} |
+----+     +-----------+     +-----------------+             +--------------------------------+
ref A          ref B                ref C                         ref E  <-- countedNames

5 steps --> 5 distinct objects. Each arrow is "read old, build new." A, B, C are discarded
right after the next box is built from them.


IMPERATIVE (reduce + mutate, or for-loop) -- ONE object, contents rewritten in place

+-----------------------------------------------------------------------------------+
| {}  >>  {Alice:1}  >>  {Alice:1,Bob:1}  >> ... >>  {Alice:2,Bob:1,Tiff:1,Bruce:1} |
+-----------------------------------------------------------------------------------+
              ref A  <-- countedNames (same reference the entire time)

5 steps --> 1 object. Each >> is "mutate contents in place." Nothing new is ever created.
```

## Three versions, compared

| Version | Reference behavior | What the construct implies | Match? | Cost | Worth it when |
|---|---|---|---|---|---|
| for-loop | same reference, mutated every step | nothing — a loop makes no promise about references | ✅ (no promise to break) | O(n) | default choice — nothing downstream needs a new reference or a history of steps |
| reduce + spread | new reference every step | "fold into a new accumulated value each step" | ✅ (keeps the promise) | O(n²) | the result feeds something that relies on reference equality (React/Redux state, `useMemo`/`useSelector`), or you need intermediate snapshots (undo history, time-travel debugging) |
| reduce + mutate | same reference, mutated every step | "fold into a new accumulated value each step" | ❌ (breaks the promise) | O(n) | common and accepted in practice — the mutation is local and contained, so it's safe; just know the syntax implies more than it delivers |

There's no free option here: cheap *and* honest isn't on the table for this operation. The for-loop is cheap because it never claimed anything. Reduce-with-spread is honest because it actually pays for a new value every step — worth it specifically when something downstream needs that new reference or that history, not as a default habit. Reduce-with-mutation is cheap by quietly not paying for what it implied, which is fine as long as nothing downstream is depending on reference changes to detect updates.

This "reference changed vs. same reference mutated" distinction is the same mechanism React, Redux, and memoization rely on for change detection — see [reference-equality](../mental-models/reference-equality.md) for where it shows up beyond plain arrays/objects.

## Proving it: reference identity

"Returning a new value" means a new *reference*, not just executing a `return` statement. Running the script below logs whether the accumulator's reference changes at each step, for all three versions:

```
--- reduce + mutate (same reference every step) ---
  step "Alice": same reference as previous? false
  step "Bob": same reference as previous? true
  step "Tiff": same reference as previous? true
  step "Bruce": same reference as previous? true
  step "Alice": same reference as previous? true
  result: [Object: null prototype] { Alice: 2, Bob: 1, Tiff: 1, Bruce: 1 }

--- for-loop (mutation is explicit, no accumulator returned each step) ---
  step "Alice"
  step "Bob"
  step "Tiff"
  step "Bruce"
  step "Alice"
  result: [Object: null prototype] { Alice: 2, Bob: 1, Tiff: 1, Bruce: 1 }

--- reduce + spread (new reference every step) ---
  step "Alice": same reference as previous? false
  step "Bob": same reference as previous? false
  step "Tiff": same reference as previous? false
  step "Bruce": same reference as previous? false
  step "Alice": same reference as previous? false
  result: { Alice: 2, Bob: 1, Tiff: 1, Bruce: 1 }
```

The first step is `false` in both reduce cases only because there's no previous reference yet to compare against. From step 2 onward is the real signal: mutate flips to `true` and stays there (same object reused every step), spread stays `false` throughout (new object every step) — same final result, different cost to get there. The mutate version returns *something* every step, so it type-checks and runs fine — but what it returns is never new. That gap between "returns a value" and "returns a new value" is invisible unless you check identity.

**Script** (copy-pasteable into `node` if you want to re-run it):

```js
const names = ["Alice", "Bob", "Tiff", "Bruce", "Alice"]

console.log("--- reduce + mutate (same reference every step) ---")
let prevMutate
const mutateResult = names.reduce((allNames, name) => {
  console.log(`  step "${name}": same reference as previous? ${prevMutate === allNames}`)
  prevMutate = allNames
  allNames[name] = (allNames[name] ?? 0) + 1
  return allNames
}, Object.create(null))
console.log("  result:", mutateResult)

console.log("\n--- for-loop (mutation is explicit, no accumulator returned each step) ---")
const loopResult = Object.create(null)
for (const name of names) {
  console.log(`  step "${name}"`)
  loopResult[name] = (loopResult[name] ?? 0) + 1
}
console.log("  result:", loopResult)

console.log("\n--- reduce + spread (new reference every step) ---")
let prevSpread
const spreadResult = names.reduce((allNames, name) => {
  console.log(`  step "${name}": same reference as previous? ${prevSpread === allNames}`)
  prevSpread = allNames
  const next = { ...allNames, [name]: (allNames[name] ?? 0) + 1 }
  return next
}, {})
console.log("  result:", spreadResult)
```

`Object.create(null)` is used for the mutate and for-loop accumulators instead of `{}` so the counter object has no prototype — no collisions with inherited keys like `toString` or `constructor`. Also worth noting: forgetting `return allNames` in the mutate version means the next iteration receives `undefined` as its accumulator — an easy mistake precisely because the mutation already "worked," masking that the return was skipped.

