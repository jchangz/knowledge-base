---
tags: [classes, oop, prototype]
date: 2026-08-07
type: pattern
---

# Instance methods vs static methods

**Interview questions this note answers:**
- Can you explain the difference between a static method and an instance method in JavaScript?
- Without using the `class` keyword, how would you implement a static method versus an instance method using ES5 constructor functions?

**TL;DR:** Does the method need `this` to be one specific object? Instance method. Doesn't need `this` at all — only whatever's explicitly passed in? Static method. Mechanically: instance methods are added to the constructor function's `.prototype` object; static methods are assigned directly as properties of the constructor function itself.

**Memory hook:** the words describe what `this` becomes at call time, not where the method is stored — both kinds of method are stored once, shared by every instance (one `sayHello` on `User.prototype`, one `compareUsers` on `User`). **static** = *stationary* (same root as "static electricity," a "static IP") — `this` never binds to any object at all. **instance** = literally *an instance of* the class — calling it binds `this` to whichever one specific object you called it on. **Static never gets a `this`; instance methods get handed a different `this` each time, even though the function itself is shared.**

| | Instance method | Static method |
|---|---|---|
| Called via | `obj.method()` | `ClassName.method()` |
| `this` refers to | the specific instance it was called on | the class it was called through |
| Access to instance data | implicit, via `this.prop` | none implicit — only data passed in explicitly as arguments |
| Typical use | behavior unique to one object | factories, comparisons/utilities across objects, checks that take an object in |

The "no access to instance data" row is easy to overstate: a static method can't reach instance data *through `this`*, but it can absolutely operate on instance data handed to it as a parameter — that's the whole point of things like `Object.hasOwn(obj, key)` or a comparison function. It's about *implicit* access, not access at all.

```js
class Circle {
  constructor(radius) { this.radius = radius; }

  area() {                              // uses `this.radius` → instance method
    return Math.PI * this.radius ** 2;
  }

  static compareArea(a, b) {            // no single circle owns this → static,
    return a.area() - b.area();         // but it still reaches into a's and b's data
  }
}
```

## Why bother putting a `this`-less function on the class at all

A static method could almost always be a free-standing function instead. Reasons to attach it to the class:

- **Namespacing** — `Object.hasOwn`, `Array.isArray`, `Math.max` avoid polluting global scope with loose function names.
- **Discoverability** — `ClassName.` in an editor surfaces everything related to that type, not just instance behavior.
- **Conceptual grouping** — documents "this is a fact/operation about `X`-ness" even when it doesn't touch `this`.
- **Inheritance with polymorphism** — the one *functional* reason, not just style. See below.

## Static methods are inherited too — through a second, separate chain

`class Admin extends User {}` wires up two prototype links, not one:

```
Admin.prototype.__proto__ === User.prototype   // instance methods delegate here
Admin.__proto__            === User             // static methods delegate here
```

Classes are objects too, so `Admin` itself has `User` as its prototype. Calling `Admin.fromJSON(...)` isn't calling a copy — JS looks on `Admin`, doesn't find it, walks up to `User`, finds it there. Same delegation mechanism as `dog.speak()` on an instance, just one level up:

```js
class User {
  constructor(name) { this.name = name; }

  static fromJSON(json) {
    return new this(JSON.parse(json).name); // `this` = whichever class it was called through
  }
}
class Admin extends User {}

Admin.hasOwnProperty('fromJSON'); // false — not actually defined on Admin
'fromJSON' in Admin;              // true  — found via the static chain

Admin.fromJSON('{"name":"Rex"}') instanceof Admin; // true, not User
```

`this` inside a static method is whichever class it was *called through*, not necessarily the class it's defined on — that's why `new this(...)` produces an `Admin`, even though the method body physically lives on `User`. This is what makes static factory methods stay correct across a class hierarchy instead of hardcoding the base class.

See [obj-key-vs-hasownproperty](obj-key-vs-hasownproperty.md) for the concrete built-in example (`Object.hasOwn` static vs `hasOwnProperty` instance) that this concept explains.

## Pre-ES6: constructor functions (what `class` compiles down to)

`class` is sugar. Before it existed (and still, under the hood), the exact same instance/static split was written by hand with a constructor function plus its `.prototype`:

```js
function User(name) {
  this.name = name;                    // instance data, set per-call via `new`
}

User.prototype.sayHello = function() {  // instance method — on the PROTOTYPE
  return `Hello, my name is ${this.name}.`;
};

User.compareUsers = function(user1, user2) { // static method — on the FUNCTION ITSELF
  return user1.name === user2.name;
};

const alice = new User('Alice');
const bob = new User('Bob');

alice.sayHello();               // looked up via alice's prototype chain → User.prototype
User.compareUsers(alice, bob);  // called directly on User, no instance involved
```

Line up the two placements against the class version: `User.prototype.sayHello` *is* `sayHello()` inside a class body, and `User.compareUsers` *is* `static compareUsers()`. `class` didn't invent a new mechanism — it just gave `.prototype.method = ...` and `Constructor.method = ...` a cleaner spelling. Seeing it written by hand like this is what makes "instance methods live on the prototype, static methods live on the constructor" concrete instead of a rule to memorize.

## When you're not using classes at all

A lot of JS never touches `class`. The instance/static *labels* disappear, but the underlying question — "does this need `this` bound to one specific piece of state?" — doesn't. It just gets answered with different tools:

- **What replaces instance methods → closures, not `this`.** A factory function that returns an object literal gets its "instance data" from variables captured in the closure, not from a `this` binding:

```js
function createCounter() {
  let count = 0;              // private "instance" data — lives in the closure, not on `this`
  return {
    increment() { count++; },
    getCount() { return count; },
  };
}

const a = createCounter();
const b = createCounter();
a.increment();
a.getCount(); // 1
b.getCount(); // 0 — separate closure, separate state, same shape as separate instances
```

- **What replaces static methods → plain exported functions.** If a method never needed `this` in the first place, there was never a reason to attach it to anything — a module's exports *are* the namespace that a class's static side would have provided:

```js
// mathUtils.js — no class needed, the module itself is the namespace
export function isEven(n) { return n % 2 === 0; }
export function max(a, b) { return a > b ? a : b; }
```

This is why plenty of style guides push back on "classes with only static methods" — if nothing in the class ever needs `this`, the class isn't doing anything a module full of exported functions wasn't already doing, minus the ceremony.

The mental shortcut carries over cleanly: **needs private/instance-specific state → reach for a closure. Doesn't need any specific state → it's just a function**, whether or not a class is anywhere in sight.
