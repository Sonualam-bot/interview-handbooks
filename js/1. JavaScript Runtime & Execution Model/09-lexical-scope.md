# Chapter 9 — Lexical Scope

> *"JavaScript doesn't decide what a function can access when it runs. It decides when the function is written."*

---

# A Mystery

Look carefully at this code.

```js
let company = "OpenAI";

function outer() {
    let team = "Platform";

    function inner() {
        console.log(company);
        console.log(team);
    }

    inner();
}

outer();
```

Why can `inner()` access both `company` and `team`?

Now imagine calling `inner()` from somewhere else.

Would it suddenly lose access to `team`?

No.

Why?

Because JavaScript made that decision long before `inner()` was ever executed.

---

# Becoming the JavaScript Engine

Imagine you're parsing the source code.

You discover:

- a global scope,
- a function named `outer`,
- another function named `inner` written inside `outer`.

Before the program even runs, you already know something important.

`inner` lives inside `outer`.

That relationship comes directly from the source code.

It never changes.

No matter where `inner` is eventually called, its surrounding environment is already fixed.

---

# What Is Lexical Scope?

**Lexical Scope** means that the visibility of variables is determined by **where code is written**, not by where it is executed.

The word *lexical* refers to the structure of the source code.

JavaScript builds the scope relationships while reading the program.

Execution simply follows those relationships.

---

# Think of a Family Tree

Imagine a family tree.

Every child permanently belongs to a parent.

Moving the child to another city doesn't change who the parent is.

Functions behave the same way.

A nested function permanently belongs to the scope where it was defined.

Calling it somewhere else doesn't rewrite its ancestry.

---

# Walking Through an Example

```js
let language = "JavaScript";

function course() {
    let topic = "Closures";

    function lesson() {
        console.log(language);
        console.log(topic);
    }

    lesson();
}
```

Conceptually:

```
Global Scope
│
└── course()
      │
      └── lesson()
```

When `lesson()` looks for `topic`, it finds it immediately.

When it looks for `language`, it doesn't find it locally, so it continues searching outward.

The search follows the lexical structure of the code.

---

# Calling Doesn't Change Scope

Consider:

```js
function outer() {
    let value = 42;

    return function inner() {
        console.log(value);
    };
}

const fn = outer();

fn();
```

Although `fn()` is called from the global scope, the function still accesses `value`.

Why?

Because `inner` was written inside `outer`.

Its lexical relationship never changed.

This single idea is the foundation of closures.

---

# Why JavaScript Uses Lexical Scope

Imagine if scope depended on where a function was called.

The same function could behave differently every time it was invoked.

Reasoning about programs would become extremely difficult.

By fixing scope when code is written, JavaScript makes functions predictable and reusable.

---

# Looking Ahead

Lexical scope explains **where JavaScript searches**.

The next question is:

> What happens if the value is still needed even after the outer function has finished?

That question leads directly to one of JavaScript's most powerful features:

**Closures.**

---

# React Connection

Every React event handler, effect, callback, and memoized function relies on lexical scope.

When an event handler reads component variables, it isn't searching randomly.

It follows the lexical relationships established when that render created the function.

Understanding lexical scope makes stale closures much easier to understand later.

---

# Key Takeaways

- Lexical scope is determined by where functions are written.
- Calling a function elsewhere does not change its scope.
- Nested functions can access identifiers from their outer lexical environments.
- JavaScript builds scope relationships before execution begins.
- Lexical scope is the foundation of closures and many React behaviors.

---

> **Next Chapter:** *Scope Chain — How JavaScript Searches for Variables*
