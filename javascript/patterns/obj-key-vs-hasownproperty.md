---
tags: [object, property-access]
date: 2026-08-06
type: pattern
---

# Checking if a key exists: `obj[key]` vs `hasOwnProperty`

**TL;DR:** Don't use `obj[key]` (or `obj.prop`) as a stand-in for "does this key exist." Use `Object.hasOwn(obj, key)` (or `obj.hasOwnProperty(key)`).

`obj[key]` answers "what's the value," not "is the key there." That distinction breaks in two ways:

**1. It reads as truthy/falsy, not present/absent.** A key holding a falsy value looks the same as a missing key:

```js
const obj = { prop: 0 };

Boolean(obj.prop);          // false — looks "missing"
obj.hasOwnProperty('prop'); // true  — but it's there
```

Same trap for `''`, `false`, `null`, `NaN` — any legitimately-falsy value.

**2. It can't tell "missing" from "present but `undefined`."** Both read as `undefined`:

```js
const obj = { prop: undefined };

obj.prop;                    // undefined — key exists, value is undefined
({}).prop;                   // undefined — key doesn't exist at all
obj.hasOwnProperty('prop');  // true, vs false on {} — this is the actual signal
```

## Prefer `Object.hasOwn` over `.hasOwnProperty`

`obj.hasOwnProperty(key)` has its own gotchas: it fails on `Object.create(null)` (no prototype → no `hasOwnProperty` method to call), and can be shadowed if the object has its own `hasOwnProperty` property. `Object.hasOwn(obj, key)` (ES2022) is a static method — see [instance-vs-static-methods](instance-vs-static-methods.md) for why that's exactly what fixes both issues — so neither applies:

```js
const noProto = Object.create(null);
noProto.prop = 1;

noProto.hasOwnProperty('prop'); // TypeError — no hasOwnProperty on the prototype chain
Object.hasOwn(noProto, 'prop'); // true — works regardless of prototype
```

**Note:** both `hasOwnProperty` and `Object.hasOwn` check *own* properties only — inherited properties (further up the prototype chain) return `false`. Use the `in` operator (`key in obj`) if you want existence checks to include inherited properties too.

```js
const obj = {};

'toString' in obj;              // true  — inherited from Object.prototype, misleading if you wanted "own"
Object.hasOwn(obj, 'toString'); // false — obj itself doesn't have it
```

`in` walks the whole prototype chain, so it'll report `true` for things like `toString` or `constructor` that every plain object inherits — rarely what you actually want when checking "did *this* object define this key."

`hasOwnProperty`/`Object.hasOwn` isn't "can I call this" — inherited methods (from a class or prototype) work fine even though they report `false`. Use `in` or just call the method if you're checking "is this usable," not "did I define this."
