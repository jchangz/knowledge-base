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

## The tradeoff

Null-prototype objects lose the built-in conveniences too — `dict.toString()` throws `TypeError`, and `console.log(dict)` prints `[Object: null prototype] {...}` as a flag that something's different. Reach for `Object.create(null)` (or a `Map`) specifically when an object is being used as a key/value store for keys you don't control — not for general-purpose objects.
