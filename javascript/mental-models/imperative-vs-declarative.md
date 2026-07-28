---
tags: [paradigms, style]
date: 2026-07-28
type: mental-model
---

# Imperative vs declarative

**Imperative** code describes *how* to do something, step by step — explicit instructions that mutate state as they go (assign a variable, loop, increment a counter).

**Declarative** code describes *what* you want and lets something else figure out the how.

## Example

```js
// imperative — explicit steps, mutates countedNames directly
const countedNames = {}
for (const name of names) {
  const currCount = countedNames[name] ?? 0
  countedNames[name] = currCount + 1
}
```

```js
// declarative — describes the transformation, no visible mutation
const countedNames = names.reduce((allNames, name) => ({
  ...allNames,
  [name]: (allNames[name] ?? 0) + 1
}), {})
```

## Why it matters

`reduce`, `map`, `filter` are designed for the declarative style — "transform this into that." When a callback passed to one of them secretly mutates state instead of returning a new value, it's imperative behavior wearing a declarative costume — see [reduce-vs-for-loop](../patterns/reduce-vs-for-loop.md) for the concrete case. The mismatch is what makes that code confusing to read, not the mutation itself.

## Other examples of the same split

- SQL (`SELECT * WHERE x > 5` — declarative) vs. a manual scan loop (imperative)
- React JSX (declare the UI for a given state) vs. manually calling `document.createElement`/`appendChild` (imperative DOM manipulation)
- CSS (declare the desired layout) vs. manually computing element positions with JS

## Takeaway

Neither style is universally "better" — declarative code tends to be easier to read and reason about because it hides mechanics you don't need to track, but it depends on the underlying tool actually doing the how correctly and efficiently. Imperative code is more verbose but leaves nothing hidden. The failure mode to watch for is mixing them: using a declarative-looking construct to do imperative work, which misleads whoever reads it next.
