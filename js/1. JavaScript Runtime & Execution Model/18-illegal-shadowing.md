# Chapter 18 — Illegal Shadowing

> *"JavaScript allows many forms of shadowing—but not all of them. Some combinations create ambiguity, so the language rejects them before your program even starts."*

---

# A Mystery

Consider these two programs.

The first works perfectly.

```js
let count = 10;

{
    let count = 20;
}

console.log(count);
```

Now look at this one.

```js
let count = 10;

{
    var count = 20;
}
```

Instead of running, JavaScript immediately reports:

```text
SyntaxError
```

Why?

If shadowing is allowed, why is this version forbidden?

---

# Becoming the JavaScript Engine

Imagine you're preparing memory.

You discover:

```js
let count = 10;
```

A binding named `count` is created in the current scope.

A little later you encounter:

```js
{
    var count = 20;
}
```

At first glance, it seems like `count` belongs to the block.

But you remember an important rule.

`var` is **not block-scoped**.

It belongs to the nearest function (or global scope).

That means this second declaration is trying to create another `count` in the **same scope**.

Two incompatible declarations now compete for the same binding.

Rather than guessing what the developer intended, JavaScript stops with a syntax error.

---

# What Is Illegal Shadowing?

**Illegal shadowing** occurs when a declaration attempts to shadow another identifier in a way that violates JavaScript's scoping rules.

Instead of producing unpredictable behavior, JavaScript rejects the program during parsing.

The code never begins execution.

---

# Why `var` Causes Trouble

Remember:

- `let` and `const` belong to blocks.
- `var` belongs to functions.

Consider:

```js
let value = 1;

{
    var value = 2;
}
```

Although `var` appears inside a block, it actually tries to create:

```js
var value = 2;
```

in the surrounding scope.

That scope already contains:

```js
let value = 1;
```

The conflict is illegal.

---

# A Legal Example

This version is perfectly valid.

```js
var total = 100;

{
    let total = 200;
}

console.log(total);
```

Why?

Because the `let` declaration belongs only to the block.

It creates a completely separate binding.

No conflict exists.

---

# Function Boundaries Matter

Now consider:

```js
let score = 10;

function play() {
    var score = 20;

    console.log(score);
}

play();

console.log(score);
```

Output:

```text
20
10
```

This is legal.

Why?

The `var` belongs to the function scope, not the global scope.

The declarations live in different lexical environments.

---

# Think of Reserved Seats

Imagine a theater.

Every seat has a unique number.

Two people cannot occupy seat **A12** in the same section.

However, another theater room can also have an **A12**.

The room separates them.

Scopes work the same way.

Different scopes may reuse names.

The same scope may not contain incompatible bindings.

---

# Why JavaScript Rejects Illegal Shadowing

Imagine allowing both declarations to coexist.

When code asks for:

```js
count
```

Which declaration should win?

The answer would depend on subtle implementation details.

JavaScript avoids this ambiguity entirely by refusing to compile the program.

---

# React Connection

Illegal shadowing is uncommon in everyday React development because modern React code primarily uses `let` and `const`.

However, understanding this rule becomes valuable when reading older codebases that still rely heavily on `var`, especially during migrations from legacy JavaScript.

---

# Key Takeaways

- Shadowing is normally allowed across different scopes.
- `var` ignores block scope and belongs to the nearest function scope.
- A `var` declaration cannot conflict with an existing `let` or `const` binding in the same scope.
- Illegal shadowing results in a `SyntaxError` before execution begins.
- Understanding scope boundaries makes these rules much easier to reason about.

---

> **Next Chapter:** *The Global Object — Why Some Variables Become Properties of `window` (or `globalThis`)*
