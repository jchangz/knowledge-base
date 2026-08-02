---
tags: [prototypes, security, objects]
date: 2026-08-01
type: gotcha
---

# Null-prototype objects

**TL;DR:** a plain `{}` isn't actually empty — it inherits every property on `Object.prototype` (`toString`, `constructor`, `__proto__`, ...) via its prototype chain. If you use a plain object as a dictionary for arbitrary/untrusted string keys, a key that collides with one of those names doesn't get a fresh slot — it shadows or triggers the inherited one. `Object.create(null)` gives you an object with no prototype chain at all, so there's nothing to collide with.

## The collision

```js
const counts = {}
counts['constructor']  // → the Object constructor function, not undefined
counts['toString']     // → a function, not undefined
```

A naive `if (counts[key])` freshness check silently lies for these keys.

## The sharper case: `__proto__`

On a normal object, `__proto__` isn't just a property name — it's an accessor inherited from `Object.prototype` that *reassigns the object's actual prototype* when set. So `obj[userControlledKey] = value`, where the key happens to be `"__proto__"`, can mutate the prototype chain instead of just storing a value. That's the classic prototype-pollution vector.

```js
const dict = Object.create(null)
dict['__proto__'] = 'x'   // just a boring own property — no accessor to trigger
dict.__proto__            // → 'x'
```

## The general principle: getters can leak references, setters can't

A single write, `obj.__proto__ = value`, is self-contained and harmless outside `obj` — the setter's contract is "reassign the receiver's own prototype," so the mutation always lands back on whoever called it, never on anything else.

The dangerous shape needs two separate steps: a **read** of a dynamic key, then a **write** on whatever that read returned.

```js
const obj = {}
const test = obj.__proto__   // GET — invokes the accessor's getter, returns Object.prototype itself, live
test.isAdmin = 2             // ordinary write — but `test` IS Object.prototype
({}).isAdmin                 // 2 — every plain object, anywhere, is now affected
```

A recursive deep-merge (`_.merge`, hand-rolled config mergers) does this by accident, one key at a time: `target[key] || {}` reads the existing nested value so it can merge into it instead of overwriting it — necessary for the merge to be "deep" at all. For every normal key that read returns `undefined` or a private nested object. For `key = '__proto__'` it returns a live reference to the shared global instead, and the merge writes into that reference believing it's still private. `Object.assign`/spread don't have this problem because they never read `target[key]` first — they only ever write once, so only the setter (self-contained) fires, never the getter (reference-leaking).

**Takeaway that generalizes past `__proto__`:** any code doing `const x = obj[dynamicKey]` and later mutating `x` is trusting that `x` is privately owned. If `dynamicKey` is attacker-influenced and can resolve to *any* accessor property (not just `__proto__` — `constructor`/`prototype` reach the same shared object by a different path), that assumption breaks silently. This is why the real fixes are structural — reject `__proto__`/`constructor`/`prototype` as keys, or remove the accessor entirely with `Object.create(null)`/`Map` — rather than "don't write recursive code."

## The tradeoff

Null-prototype objects lose the built-in conveniences too — `dict.toString()` throws `TypeError`, and `console.log(dict)` prints `[Object: null prototype] {...}` as a flag that something's different. Reach for `Object.create(null)` (or a `Map`) specifically when an object is being used as a key/value store for keys you don't control — not for general-purpose objects.
