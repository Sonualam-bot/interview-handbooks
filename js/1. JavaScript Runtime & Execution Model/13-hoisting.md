# Chapter 13 — Hoisting

> *"Nothing in JavaScript actually moves to the top of your file. The illusion comes from something much deeper."*

---

# A Mystery

Consider this code.

```js
console.log(user);

var user = "Sonu";
```

The output is:

```text
undefined
```

Now look at this.

```js
greet();

function greet() {
    console.log("Hello!");
}
```

It works perfectly.

But this one does not.

```js
sayHi();

const sayHi = function () {
    console.log("Hi!");
};
```

Instead, JavaScript throws an error.

Why do these three examples behave differently?

If JavaScript executes code from top to bottom, how can it use declarations that appear later?

Many developers answer:

> "Because JavaScript moves declarations to the top."

That explanation is simple.

It is also wrong.

---

# Becoming the JavaScript Engine

Imagine you're about to execute a JavaScript file.

Do you immediately start with line one?

No.

As we learned in the **Memory Creation Phase**, you first scan the entire scope.

While scanning, you discover:

- variable declarations,
- function declarations,
- parameters,
- block declarations.

You prepare memory for all of them before a single statement executes.

Nothing has moved.

You simply already know what exists.

---

# The Illusion of Movement

Imagine reading the table of contents of a book before reading Chapter 1.

Later, when someone asks:

> "Does Chapter 7 exist?"

You can immediately answer yes.

Not because Chapter 7 moved to the beginning.

Because you already discovered it during preparation.

Hoisting works the same way.

JavaScript doesn't rearrange your code.

It prepares declarations before execution begins.

---

# What Is Hoisting?

**Hoisting** is the observable behavior that declarations appear to be available before their position in the source code.

The behavior is a consequence of the Memory Creation Phase.

No source code is physically relocated.

---

# `var` Hoisting

Consider:

```js
console.log(score);

var score = 100;
```

During memory creation:

```
score → undefined
```

Execution begins.

The first statement reads `score`.

Since the binding already exists, JavaScript returns:

```text
undefined
```

Later:

```js
score = 100;
```

The value is assigned.

The declaration was prepared early.

The assignment still happened exactly where it was written.

---

# Function Declaration Hoisting

Now consider:

```js
greet();

function greet() {
    console.log("Hello");
}
```

During memory creation, JavaScript creates the entire function object.

Conceptually:

```
greet → Function Object
```

So when execution reaches:

```js
greet();
```

the function already exists.

That's why function declarations can be invoked before they appear in the file.

---

# Function Expressions

Now compare:

```js
sayHi();

const sayHi = function () {
    console.log("Hi");
};
```

The identifier `sayHi` exists during memory creation.

However, it does **not** receive the function object yet.

The function expression is evaluated only during execution.

At the time `sayHi()` is called, the assignment hasn't happened.

The function simply doesn't exist yet.

---

# Hoisting Isn't Execution

One of the biggest misconceptions is believing hoisting executes code early.

It doesn't.

Consider:

```js
var total = calculate();
```

JavaScript does **not** execute `calculate()` during memory creation.

Only the declaration is prepared.

Every expression, assignment, and function call still executes from top to bottom during the Execution Phase.

---

# Why JavaScript Does This

Imagine discovering a variable for the first time halfway through execution.

The engine would need to stop, reorganize memory, and update internal structures.

Preparing declarations beforehand makes execution simpler and more predictable.

Hoisting isn't a feature added for developers.

It's a consequence of how the engine prepares execution.

---

# React Connection

Every React render starts with a fresh Memory Creation Phase.

Function declarations inside a component are prepared before execution begins.

Variables declared with `let` and `const` are also created, but they aren't usable until execution reaches their declarations.

Understanding hoisting helps explain why helper functions often work before they appear, while arrow-function assignments do not.

---

# Key Takeaways

- Hoisting is a consequence of the Memory Creation Phase.
- JavaScript does **not** move code.
- `var` declarations are initialized with `undefined`.
- Function declarations are fully created before execution begins.
- Function expressions are not available until their assignment executes.
- Hoisting prepares declarations, not statements.

---

> **Next Chapter:** *The Temporal Dead Zone — Why `let` and `const` Behave Differently from `var`*
