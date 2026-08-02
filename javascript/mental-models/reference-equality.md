---
tags: [reference-equality, react, redux, memoization]
date: 2026-07-28
type: mental-model
---

# Reference equality as change detection

**TL;DR:** most JS tooling checks "did anything change" the cheap way — comparing references (`===`) instead of actually inspecting the values. Mutate an object in place and its reference never changes, so anything relying on reference equality thinks nothing happened, even though the contents did. A bunch of rules that feel like style preferences — don't mutate state, don't mutate a reducer's state — are really just this one fact showing up again and again.

See [reduce-vs-for-loop](../patterns/reduce-vs-for-loop.md) for a worked example of the same reference-vs-mutation distinction in plain JS, without any framework involved. See [shared-mutable-references](shared-mutable-references.md) for how this same fact shows up at other scales, including prototype pollution.

## The fundamental case

```js
console.log({} === {}) // false — different references, even though both are "empty objects"

const a = {}
const b = a
b.foo = 1
console.log(a === b) // true — same reference, `a` sees the mutation too
```

This is the whole mechanism. Everything below is this same fact, applied inside a specific tool.

## Where it shows up

**React state.** `useState`/`setState` decide whether to re-render by comparing old and new state with reference equality (`Object.is`).

```js
// doesn't re-render — same reference, React sees no change
state.items.push(x)
setState(state)

// re-renders — new reference
setState({ ...state, items: [...state.items, x] })
```

This is one of the most common real-world React bugs and a standard interview question ("why doesn't my UI update when I push to an array in state?").

**Redux reducers.** Must return new state, never mutate, for the same reason — `react-redux`'s `useSelector` (and connected components generally) skip re-renders when the reference is unchanged. The honest answer to "why must reducers be pure/immutable" is this mechanism, not "mutation is bad practice" in the abstract.

**`useMemo`/`useCallback`/`useEffect` dependency arrays.** These also compare dependencies by reference.

- Mutate an object that's in a dependency array → the effect/memo won't re-run when it should (the reference "looks" unchanged).
- Construct a brand new object every render and put it in a dependency array → the effect/memo re-runs every render even when nothing meaningfully changed (the reference is "new" every time, even if the values are identical).

Both are extremely common sources of "why is my effect firing every render" / "why isn't my memoized value updating" bugs.

**Immer** (used inside Redux Toolkit) is the practical resolution to the cost tradeoff in [reduce-vs-for-loop](../patterns/reduce-vs-for-loop.md#three-versions-compared): it lets you write code that *looks* like it's mutating a draft, then produces a genuinely new reference underneath using structural sharing — so you get the honesty of a new reference without paying full copy cost for the whole structure. If Immer or Redux Toolkit comes up, this mental model is the reason it exists.
