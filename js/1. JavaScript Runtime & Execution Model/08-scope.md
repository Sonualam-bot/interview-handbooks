# Chapter 8 — Scope

> *"Owning a variable is one thing. Being allowed to access it is another. JavaScript enforces rules that decide who can see what."*

---

# A Mystery

Consider this program.

```js
let company = "OpenAI";

function employee() {
    let name = "Sonu";

    console.log(company);
}

employee();

console.log(name);
```

The first `console.log()` works perfectly.

The second throws:

```text
ReferenceError: name is not defined
```

Why?

The variable clearly exists.

JavaScript created memory for it.

So why can't the global code access it?

The answer lies in **Scope**.

---

# Becoming the JavaScript Engine

Imagine you've created two Execution Contexts.

The Global Execution Context owns:

```text
company
```

The `employee()` Execution Context owns:

```text
name
```

Now you're asked to execute:

```js
console.log(name);
```

from the Global Execution Context.

Should you allow it?

If every piece of code could freely access every variable in every function, there would be no privacy, no isolation, and no predictable behavior.

So JavaScript introduces rules.

Not every identifier is visible everywhere.

---

# What Is Scope?

**Scope** is the set of rules that determines where an identifier can be accessed in a program.

Scope answers questions like:

- Can this function access that variable?
- Can this block access that constant?
- If multiple variables have the same name, which one should JavaScript use?

Without scope, large applications would quickly become impossible to reason about.

---

# Think of Rooms in a House

Imagine a house.

The living room is accessible to everyone in the house.

A private bedroom is only accessible to the person who owns it.

A locker inside that bedroom is even more restricted.

Variables behave similarly.

Some are visible almost everywhere.

Others are private to a function or block.

Scope defines those boundaries.

---

# Global Scope

Variables declared outside every function belong to the Global Scope.

```js
const appName = "Dashboard";
```

Any code that can reach the global environment can access `appName`.

```js
function show() {
    console.log(appName);
}
```

The function can read it because global scope is visible from inside the function.

---

# Function Scope

Variables declared inside a function belong only to that function.

```js
function login() {
    let token = "abc123";
}
```

Outside the function:

```js
console.log(token);
```

JavaScript cannot find the identifier.

The variable exists only within the function's scope.

---

# Block Scope

JavaScript also creates scope for blocks.

```js
if (true) {
    let age = 26;
    const city = "Bangalore";
}
```

Outside the block:

```js
console.log(age);
```

results in:

```text
ReferenceError
```

`let` and `const` respect block boundaries.

---

# A Walk Through an Example

```js
let company = "OpenAI";

function developer() {
    let name = "Sonu";

    if (true) {
        let skill = "React";

        console.log(company);
        console.log(name);
        console.log(skill);
    }
}
```

Inside the block:

- `skill` is visible.
- `name` is visible.
- `company` is visible.

The innermost scope can access identifiers from its outer scopes.

But the reverse is not true.

---

# Why Scope Exists

Imagine a program with thousands of variables.

If every variable were globally accessible:

- unrelated code could overwrite values,
- accidental name collisions would become common,
- debugging would become extremely difficult.

Scope gives every piece of code a well-defined boundary.

It protects local state from unrelated parts of the program.

---

# Scope Is Determined by Where Code Is Written

One of JavaScript's most important ideas is that scope depends on **where functions are written**, not where they are called.

This is called **lexical scope**.

We'll explore it in depth in the next chapter.

---

# React Connection

Every React component relies heavily on scope.

Local variables declared inside a component are only available during that component's execution.

Variables declared inside an event handler are visible only within that handler.

Understanding scope is essential before learning closures, hooks, and stale state.

---

# Key Takeaways

- Scope defines where identifiers are accessible.
- Global scope is available throughout the program.
- Function scope limits access to a single function.
- Block scope limits access to a block when using `let` and `const`.
- Inner scopes can access outer scopes, but outer scopes cannot access inner scopes.

---

> **Next Chapter:** *Lexical Scope — Why Where You Write a Function Matters More Than Where You Call It*
