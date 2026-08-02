---
tags: [meta, interview-prep]
date: 2026-08-02
type: meta
---

# Interview questions to be ready for

Pulled from the reference-equality / prototype-pollution deep dive. Goal: be able to answer these from the mechanism, not a memorized rule. Check off once you can explain one out loud without notes.

## Near-certain (core JS/React staples)

- [ ] Why does `{} === {}` evaluate to `false`?
- [ ] Why doesn't React re-render when you mutate state directly (`state.items.push(x)`) instead of replacing it?
- [ ] When would you use `Map` instead of a plain object, and why?
- [ ] Why must Redux reducers avoid mutating state — what actually breaks if they do?

→ [reference-equality](../javascript/mental-models/reference-equality.md), [shared-mutable-references](../javascript/mental-models/shared-mutable-references.md)

## Plausible (JS-fundamentals rounds, mid/senior)

- [ ] What is the prototype chain? How does inheritance work in JS without classes?
- [ ] What's the difference between `__proto__` and `.prototype`?
- [ ] What does `Object.create(null)` do, and when would you reach for it?
- [ ] What's the difference between a getter/setter accessor property and a normal data property?
- [ ] How would you implement a simple reactive/observable object using getters/setters? (relevant to explaining how Vue/MobX reactivity works internally)

→ [null-prototype-objects](../javascript/mental-models/null-prototype-objects.md)

## Niche (security/appsec-leaning, or senior "tell me about an interesting bug" moments)

- [ ] What is prototype pollution? Walk through a concrete example.
- [ ] Why does a recursive deep-merge leak a reference to `Object.prototype`, but `Object.assign`/spread don't?
- [ ] How would you defend a deep-merge function against this class of bug?

→ [null-prototype-objects](../javascript/mental-models/null-prototype-objects.md)
