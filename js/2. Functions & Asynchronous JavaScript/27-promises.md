# Chapter 27 — Promises

> *"Callbacks gave JavaScript the power to work asynchronously. Promises gave developers a sane way to reason about it."*

---

# A Mystery

Consider this code.

```js
fetch("/users")
  .then(response => response.json())
  .then(data => {
    console.log(data);
  })
  .catch(error => {
    console.error(error);
  });
```

Why doesn't `fetch()` return the user data immediately?

If JavaScript executes one line at a time, how can code continue running while the request is still in progress?

And what exactly is being returned from `fetch()`?

---

# Becoming the JavaScript Engine

You encounter:

```js
const result = fetch("/users");
```

The network request may take milliseconds—or seconds.

You cannot stop the entire JavaScript engine and wait.

Instead, you immediately create a special object.

```text
Promise
```

This object represents work whose result is **not available yet**.

JavaScript continues executing the rest of the program while the runtime performs the network request.

When the request finishes, the Promise changes state and schedules the appropriate callbacks.

---

# What Is a Promise?

A **Promise** is an object that represents the eventual completion or failure of an asynchronous operation.

Think of it as a receipt.

You don't have the final product yet, but you have proof that work is in progress and a way to receive the result later.

---

# Promise States

A Promise always exists in one of three states.

## Pending

The operation is still running.

```text
Promise
↓

Pending
```

---

## Fulfilled

The operation completed successfully.

```text
Promise
↓

Fulfilled

↓

Value Available
```

---

## Rejected

The operation failed.

```text
Promise
↓

Rejected

↓

Reason Available
```

Once a Promise becomes fulfilled or rejected, its state never changes again.

---

# Consuming a Promise

Instead of blocking execution, you register callbacks.

```js
fetch("/users")
  .then(data => {
    console.log(data);
  })
  .catch(error => {
    console.error(error);
  });
```

- `.then()` handles success.
- `.catch()` handles failure.
- `.finally()` runs regardless of the outcome.

These callbacks are executed later as **microtasks**.

---

# Promise Chaining

One Promise can feed another.

```js
fetch("/users")
  .then(response => response.json())
  .then(users => users[0])
  .then(user => console.log(user.name));
```

Each `.then()` returns a new Promise.

This allows asynchronous operations to be expressed as a sequence instead of deeply nested callbacks.

---

# Solving Callback Hell

Instead of:

```js
login(() => {
  fetchProfile(() => {
    fetchPosts(() => {
      // ...
    });
  });
});
```

You can write:

```js
login()
  .then(fetchProfile)
  .then(fetchPosts)
  .catch(handleError);
```

The flow becomes linear, easier to read, and easier to maintain.

---

# A Real-World Analogy

Imagine ordering a custom laptop online.

You don't wait at the factory.

Instead, you receive an order confirmation.

While the laptop is being assembled, you continue with your day.

Later, the company notifies you that:

- your order shipped,
- or there was a problem.

A Promise works the same way.

---

# React Connection

Promises are everywhere in React applications.

Examples include:

- fetching API data,
- authentication,
- lazy loading,
- server communication,
- data fetching libraries like React Query.

Even though modern React often uses `async`/`await`, those features are built directly on top of Promises.

---

# Key Takeaways

- A Promise represents future completion or failure.
- Promises have three states: Pending, Fulfilled, and Rejected.
- `.then()`, `.catch()`, and `.finally()` register callbacks.
- Promise callbacks execute as microtasks.
- Promises solve many readability problems caused by nested callbacks.

---

> **Next Chapter:** *Async/Await — Writing Asynchronous Code That Looks Synchronous*
