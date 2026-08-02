---
tags: [references, mutation, immutability, ecosystem, hub]
date: 2026-08-02
type: mental-model
---

# Shared mutable references (the throughline)

**TL;DR:** one fact — *objects are shared by reference, so mutating an object mutates it for every variable/scope holding that same reference* — is the root cause behind a surprising spread of things that look unrelated: React not re-rendering, a prototype-pollution CVE, and why `Map`/Immer/immutability conventions exist at all. Different scale, same rule. This note is the map; the linked notes have the depth.

## The one fact, at three scales

| Scale | What's shared | What breaks when you mutate it | Note |
|---|---|---|---|
| **App state** | one object referenced from two places (`state.items` held by React, mutated via `.push`) | tools that check `===` to detect change (React, Redux, `useMemo` deps) see no change, even though the contents changed | [reference-equality](reference-equality.md) |
| **The language's object model** | `Object.prototype`, the one prototype every plain object inherits from | mutate it (accidentally, via `__proto__`) and *every object in the process* — not just yours — picks up the change | [null-prototype-objects](null-prototype-objects.md) |
| **A single dictionary object** | inherited names (`toString`, `constructor`, `__proto__`) colliding with your own dynamic keys | a "fresh" key isn't actually fresh — it silently shadows or triggers something inherited | [null-prototype-objects](null-prototype-objects.md) |

## Why the ecosystem keeps building safer primitives on top

Once you see the one fact, these all read as the same fix applied in different places:

- **`Map` over `{}` for dictionaries** — sidesteps inherited-key collisions entirely; never shares anything with `Object.prototype`.
- **Redux/reducer purity ("never mutate state")** — not a style rule, it's making sure `===` checks stay honest. See [reference-equality](reference-equality.md).
- **Immer** — lets you *write* mutation-style code but produces a genuinely new reference underneath, so you get both ergonomics and honest change-detection.
- **`Object.create(null)` for merge targets** — removes the shared root (`Object.prototype`) from the chain entirely, so there's nothing global left to leak a reference to.
- **Explicit field assignment over generic recursive merge** — never reads a dynamic key at all, so it can never receive a reference to something it didn't expect.

## The one question that catches most of this in review

*"Am I about to mutate something I only meant to read, or hand a reference to something I don't privately own?"*

- `state.items.push(x)` — pushed into a reference React also holds → stale UI.
- `target[key] || {}` where `key` is untrusted → might return `Object.prototype` itself → global pollution.
- `obj['someKey']` where `someKey` collides with an inherited name → silently not what you think.

Same question, three different bugs.
