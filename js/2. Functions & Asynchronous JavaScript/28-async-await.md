# Chapter 28 — Async/Await

> *"Promises made asynchronous code easier to manage. `async` and `await` made it feel almost like ordinary synchronous code—without changing how JavaScript actually works."*

---

# A Mystery

Look at this code.

```js
async function loadUser() {
    const response = await fetch("/users");
    const users = await response.json();

    console.log(users);
}
```

It almost looks like JavaScript pauses at each `await`.

Does the entire JavaScript engine stop?

Can other code execute while this function is waiting?

If JavaScript is single-threaded, how can this possibly work?

---

# Becoming the JavaScript Engine

You encounter:

```js
await fetch("/users");
```

The `fetch()` call immediately returns a Promise.

Instead of blocking the Call Stack, you suspend **only the current async function**.

Conceptually:

```text
Current async function

↓

Paused

↓

Call Stack becomes available

↓

Other JavaScript continues running
```

When the Promise settles, the suspended function is scheduled to continue as a **microtask**.

---

# What Does `async` Mean?

Adding the `async` keyword changes the behavior of a function.

```js
async function greet() {
    return "Hello";
}
```

Even though it returns a string, JavaScript automatically wraps it in a Promise.

Conceptually:

```js
Promise.resolve("Hello")
```

So:

```js
const result = greet();
```

returns a Promise, not a plain string.

---

# What Does `await` Mean?

`await` can only be used inside an `async` function (or at the top level in supported modules).

```js
const data = await fetch("/users");
```

It means:

> "Pause this async function until the Promise settles, then continue with its result."

Importantly, it does **not** pause the rest of the program.

---

# Error Handling

With Promises, errors are commonly handled using `.catch()`.

```js
fetch("/users")
    .catch(error => {
        console.error(error);
    });
```

With `async`/`await`, normal `try...catch` works naturally.

```js
async function loadUser() {
    try {
        const response = await fetch("/users");
        console.log(await response.json());
    } catch (error) {
        console.error(error);
    }
}
```

This makes asynchronous error handling look very similar to synchronous code.

---

# Why Was `async`/`await` Introduced?

Promise chains are much cleaner than callback hell, but long chains can still become difficult to read.

```js
login()
    .then(fetchProfile)
    .then(fetchPosts)
    .then(renderPosts)
    .catch(handleError);
```

The same flow with `async`/`await`:

```js
async function start() {
    try {
        const profile = await fetchProfile();
        const posts = await fetchPosts(profile);

        renderPosts(posts);
    } catch (error) {
        handleError(error);
    }
}
```

The control flow becomes easier to follow from top to bottom.

---

# A Real-World Analogy

Imagine ordering coffee.

You place your order and receive a token.

Instead of standing at the counter doing nothing, you find a seat and continue reading a book.

When your number is called, you return to collect the coffee.

`await` behaves similarly.

The function waits.

The rest of the world continues moving.

---

# Common Misconception

Many developers believe:

> "`await` blocks JavaScript."

It does not.

It only suspends the current async function.

Everything else—including event handlers, timers, rendering, and other asynchronous work—continues normally.

---

# React Connection

Modern React applications use `async`/`await` constantly.

Examples include:

```js
const response = await fetch("/api/products");
```

```js
const data = await response.json();
```

```js
await mutation.mutateAsync(values);
```

Although the syntax appears synchronous, React still relies on Promises, microtasks, and the Event Loop underneath.

Understanding these lower-level concepts makes debugging asynchronous React code much easier.

---

# Key Takeaways

- `async` functions always return Promises.
- `await` pauses only the current async function.
- The JavaScript engine remains free to execute other work.
- `try...catch` works naturally with `async`/`await`.
- `async`/`await` is built entirely on top of Promises.

---

> **Next Chapter:** *Fetch API — How JavaScript Communicates with Servers*
