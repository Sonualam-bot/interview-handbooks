# Chapter 15 — `var`, `let`, and `const`

> *"JavaScript has three ways to declare variables. They all create bindings—but they don't all follow the same rules."*

---

# A Mystery

Consider these three declarations.

```js
var username = "Sonu";

let city = "Bangalore";

const country = "India";
```

At first glance, they appear interchangeable.

All three create variables.

All three can store values.

All three seem to work.

So why does JavaScript have three different keywords?

The answer lies in the language's evolution.

---

# A Brief Journey Through Time

When JavaScript was created in 1995, there was only one way to declare variables:

```js
var
```

For years, every variable used `var`.

As applications became larger, developers discovered problems:

- accidental global variables,
- variables escaping blocks,
- confusing hoisting behavior,
- silent bugs caused by reassignment.

When ECMAScript 2015 (ES6) arrived, JavaScript introduced two new declarations:

```js
let
const
```

These weren't added to replace `var` completely.

They were designed to solve problems that `var` could not.

---

# Becoming the JavaScript Engine

Imagine you're preparing memory.

You encounter:

```js
var score;
let level;
const pi = 3.14;
```

You don't treat them equally.

For `var`:

- create the binding,
- initialize it with `undefined`.

For `let`:

- create the binding,
- leave it uninitialized until execution reaches the declaration.

For `const`:

- create the binding,
- also leave it uninitialized,
- require a value during initialization.

The keyword determines the rules attached to the binding.

---

# `var`

```js
var count = 10;
```

Characteristics:

- function-scoped,
- initialized with `undefined`,
- can be reassigned,
- can be redeclared in the same scope.

```js
var age = 20;
var age = 25;

console.log(age);
```

Output:

```text
25
```

---

# `let`

```js
let count = 10;
```

Characteristics:

- block-scoped,
- exists in the Temporal Dead Zone until initialized,
- can be reassigned,
- cannot be redeclared in the same scope.

```js
let score = 50;

score = 75;
```

Perfectly valid.

But:

```js
let score = 50;
let score = 75;
```

results in a syntax error.

---

# `const`

```js
const PI = 3.14159;
```

Characteristics:

- block-scoped,
- participates in the Temporal Dead Zone,
- cannot be redeclared,
- cannot be reassigned after initialization.

```js
const PORT = 3000;

PORT = 4000;
```

Results in:

```text
TypeError
```

---

# Does `const` Make Objects Immutable?

Consider:

```js
const user = {
    name: "Sonu"
};

user.name = "Alex";
```

This works.

Why?

Because `const` protects the **binding**, not the object.

The variable must continue referring to the same object.

The object's internal properties may still change.

---

# A Quick Comparison

| Feature | `var` | `let` | `const` |
|---------|-------|-------|---------|
| Scope | Function | Block | Block |
| Hoisted | Yes | Yes | Yes |
| TDZ | No | Yes | Yes |
| Initialized during Memory Creation | `undefined` | No | No |
| Reassignable | ✅ | ✅ | ❌ |
| Redeclarable | ✅ | ❌ | ❌ |

---

# Which One Should You Use?

Modern JavaScript generally follows a simple guideline.

Use:

- `const` by default.
- `let` when reassignment is required.
- `var` only when maintaining or understanding older codebases.

This convention reduces accidental mutations and makes intent clearer.

---

# React Connection

Most React codebases almost exclusively use `const` and `let`.

Components, hooks, helper functions, JSX values, and imports are commonly declared with `const`.

State variables change through React's APIs—not by reassigning local bindings—making `const` a natural fit.

---

# Key Takeaways

- `var`, `let`, and `const` all create bindings, but with different rules.
- `var` is function-scoped and initialized with `undefined`.
- `let` is block-scoped and can be reassigned.
- `const` is block-scoped and cannot be reassigned.
- `const` protects the binding, not the contents of an object.

---

> **Next Chapter:** *Block Scope — Why Curly Braces Create New Worlds in JavaScript*
