# Chapter 19 — The Global Object

> *"Before your code creates its first variable, JavaScript has already created an object that represents the entire runtime environment."*

---

# A Mystery

Consider this code running in a browser.

```js
var appName = "Dashboard";

console.log(window.appName);
```

Output:

```text
Dashboard
```

Now compare it with:

```js
let version = "1.0";

console.log(window.version);
```

Output:

```text
undefined
```

Both declarations are global.

So why does only one become a property of `window`?

---

# Becoming the JavaScript Engine

Imagine you're creating the Global Execution Context.

Before reading the user's code, you already create a special object.

In browsers, that object is `window`.

In modern JavaScript, the standard way to refer to it is:

```js
globalThis
```

This object already contains hundreds of built-in APIs:

- `console`
- `setTimeout`
- `fetch`
- `Math`
- `JSON`

Only after this environment exists do you begin processing the user's declarations.

---

# What Is the Global Object?

The **Global Object** is a special object automatically created by the JavaScript runtime.

It provides globally available APIs and acts as the root object for the environment.

Its exact name depends on where JavaScript runs.

| Environment | Global Object |
|-------------|---------------|
| Browser | `window` |
| Web Worker | `self` |
| Node.js | `global` |
| Standard cross-platform reference | `globalThis` |

Today, `globalThis` is the recommended way to access the global object because it works consistently across environments.

---

# Think of an Operating System

Imagine turning on a computer.

Before you launch your first application, the operating system has already loaded services like:

- networking,
- file management,
- memory management,
- the desktop.

Your program starts inside an environment that already exists.

The Global Object plays a similar role.

It provides the environment in which your JavaScript code executes.

---

# Why Does `var` Become a Property?

Consider:

```js
var username = "Sonu";
```

Historically, global `var` declarations become properties of the global object.

Conceptually:

```text
window.username

↓

"Sonu"
```

or more generally:

```js
globalThis.username
```

This behavior exists for historical compatibility with early JavaScript.

---

# Why Don't `let` and `const`?

Now consider:

```js
let city = "Bangalore";
const country = "India";
```

These declarations are still global.

However, they create bindings in the Global Lexical Environment—not properties on the Global Object.

Therefore:

```js
globalThis.city
```

returns:

```text
undefined
```

The variables exist.

They simply aren't stored as object properties.

---

# Built-in Globals

Many things that seem like language features are actually available through the Global Object.

Examples include:

```js
setTimeout(...)
clearInterval(...)
fetch(...)
console.log(...)
```

These functions are provided by the runtime environment, not by the JavaScript language itself.

---

# Why `globalThis` Was Introduced

Before `globalThis`, writing universal JavaScript required checking the environment.

```js
window
global
self
```

Each platform used a different global object.

`globalThis` provides a single standard reference regardless of where the code runs.

---

# React Connection

React applications running in the browser often interact with browser globals such as:

```js
window
document
localStorage
navigator
```

When using React frameworks like Next.js, remembering that these globals only exist in browser environments helps explain why code using `window` cannot run during server-side rendering.

Using `globalThis` is appropriate when writing environment-independent JavaScript, though browser-specific APIs still require a browser runtime.

---

# Key Takeaways

- Every JavaScript runtime creates a Global Object before your code executes.
- `window`, `global`, and `self` are environment-specific global objects.
- `globalThis` is the standard cross-platform reference.
- Global `var` declarations become properties of the Global Object.
- Global `let` and `const` declarations do not become Global Object properties.
- Many familiar APIs are exposed through the Global Object.

---

> **Next Chapter:** *this — Understanding JavaScript's Most Misunderstood Keyword*
