---
tags: [paradigms, style]
date: 2026-07-28
type: mental-model
---

# Imperative vs declarative

**TL;DR:** Imperative code spells out the steps — loop, mutate, assign, repeat. Declarative code just describes the result you want and lets something else handle the how (`reduce`, SQL, JSX). Neither is automatically better: declarative reads cleaner, but only if the thing doing the "how" for you actually does it well; imperative is more verbose but hides nothing. The real trap is mixing them — using something that *looks* declarative (like `reduce`) to secretly do imperative work. That's what confuses the next reader, not the mutation itself.

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

## How to remember which is which

Ordering food: **imperative** is walking into the kitchen and telling the cook each step ("heat the pan, crack the egg, flip it now") — you own every instruction. **Declarative** is sitting at the table and saying "I'll have the omelette" — you state the result, the kitchen owns the how.
