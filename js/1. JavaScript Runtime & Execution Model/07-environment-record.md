# Chapter 7 — Environment Record

> *"A library is useless without shelves. A Lexical Environment is useless without a place to store its identifiers."*

---

# A Mystery

You already know that every Execution Context owns a Lexical Environment.

But here's a new question.

Suppose JavaScript executes:

```js
let name = "Sonu";
const age = 26;

function greet(message) {
    return message + ", " + name;
}
```

Where exactly are these identifiers stored?

How does the engine know that `name` maps to `"Sonu"` and `greet` maps to a function?

There must be a structure inside the Lexical Environment responsible for keeping these bindings organized.

---

# Becoming the JavaScript Engine

Imagine you're executing a function.

The first declaration appears:

```js
let count = 0;
```

You need to remember two things:

- the identifier: `count`
- its current value: `0`

Then another declaration appears.

```js
const limit = 10;
```

Then a parameter.

```js
function add(a, b) {}
```

Then another function.

Very quickly you have dozens of identifiers.

Searching randomly every time would be slow.

You need a catalog.

---

# The Filing Cabinet

Think of the Lexical Environment as an office.

Inside that office is a filing cabinet.

Every drawer has two pieces of information:

- a label (the identifier)
- what that identifier currently refers to

Examples:

```
name  → "Sonu"

age   → 26

greet → Function Object
```

That filing cabinet is called the **Environment Record**.

---

# What Is an Environment Record?

An Environment Record is the internal structure inside a Lexical Environment that stores bindings between identifiers and their values.

Whenever JavaScript needs a variable, parameter, constant, or function declaration, it first looks inside the current Environment Record.

---

# A Walk Through an Example

```js
let company = "OpenAI";

function employee(role) {
    const name = "Sonu";
}
```

Conceptually:

Global Environment Record

```
company → "OpenAI"

employee → Function Object
```

When `employee("Frontend")` executes, JavaScript creates a new Environment Record.

```
role → "Frontend"

name → "Sonu"
```

Notice that the local Environment Record does not replace the global one.

It exists alongside it.

Every execution gets its own independent record.

---

# Why Parameters Live Here

Parameters are simply identifiers whose values are supplied by the caller.

```js
function multiply(a, b) {
    return a * b;
}
```

When invoked as:

```js
multiply(3, 4);
```

the Environment Record conceptually becomes:

```
a → 3

b → 4
```

From the engine's point of view, parameters and variables are treated similarly—they are both bindings.

---

# Why This Separation Matters

Imagine if every function shared one giant table of identifiers.

Two different functions could both declare:

```js
let count = 0;
```

Without separate Environment Records, one declaration could overwrite the other.

By creating a new record for every execution, JavaScript guarantees isolation.

---

# Looking Ahead

The Environment Record answers:

> **Where are identifiers stored?**

The next question is different.

> **Who is allowed to access those identifiers?**

That question leads us to **Scope**.

---

# React Connection

Every React render creates a fresh Environment Record.

Local variables, helper functions, event handlers, and values returned by hooks are all represented by bindings inside that render's Environment Record.

A later render receives a completely new record, which is why renders remain isolated.

---

# Key Takeaways

- Every Lexical Environment contains an Environment Record.
- The Environment Record stores identifier-to-value bindings.
- Variables, constants, parameters, and function declarations all become bindings.
- Every function invocation creates a brand-new Environment Record.
- The next chapter explains how JavaScript decides which code can access those bindings.

---

> **Next Chapter:** *Scope — The Rules That Decide Who Can Access a Variable*
