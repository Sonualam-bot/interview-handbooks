# Chapter 26 — Microtasks vs Macrotasks

> *"Not all asynchronous work is treated equally. Some callbacks are so important that JavaScript always lets them cut to the front of the line."*

---

# A Mystery

Look at this code.

```js
console.log("Start");

setTimeout(() => {
    console.log("Timer");
}, 0);

Promise.resolve().then(() => {
    console.log("Promise");
});

console.log("End");
```

What will be printed?

Many people guess:

```text
Start
End
Timer
Promise
```

But the actual output is:

```text
Start
End
Promise
Timer
```

Why does the Promise callback run first when both are asynchronous?

---

# Becoming the JavaScript Engine

You execute:

```js
console.log("Start");
```

Then:

```js
setTimeout(...)
```

The runtime starts the timer. The callback will eventually enter the **Macrotask Queue**.

Next:

```js
Promise.resolve().then(...)
```

The Promise is already resolved.

Its callback is immediately placed into the **Microtask Queue**.

Finally:

```js
console.log("End");
```

The Call Stack is now empty.

The Event Loop checks the queues.

Before touching the Macrotask Queue, it completely empties the Microtask Queue.

Only then does it execute the timer callback.

---

# Two Different Queues

JavaScript schedules asynchronous work using different queues.

## Microtask Queue

Used for work that should happen **as soon as the current synchronous code finishes**.

Examples:

- `Promise.then()`
- `Promise.catch()`
- `Promise.finally()`
- `queueMicrotask()`
- `MutationObserver`

---

## Macrotask Queue

Used for work that can wait until the next Event Loop cycle.

Examples:

- `setTimeout()`
- `setInterval()`
- DOM events
- MessageChannel
- I/O callbacks (Node.js)

---

# The Event Loop's Priority

The Event Loop follows this pattern:

1. Execute all synchronous code.
2. Empty the entire Microtask Queue.
3. Execute one Macrotask.
4. Repeat.

Conceptually:

```text
Call Stack
    ↓
Microtasks
    ↓
One Macrotask
    ↓
Repeat
```

This priority guarantees that Promise reactions happen before timers.

---

# Why This Design?

Imagine you're writing a Promise chain.

```js
fetchData()
  .then(processData)
  .then(updateUI);
```

You expect each step to happen immediately after the previous one completes.

If timer callbacks could interrupt this chain, programs would become harder to reason about.

Giving microtasks higher priority keeps Promise-based workflows predictable.

---

# A Real-World Analogy

Imagine you're at an airport.

Passengers are waiting in the general boarding line.

Suddenly, a flight attendant announces:

> "Families with infants may board first."

Those passengers don't skip security.

They simply receive priority once boarding begins.

Microtasks work the same way.

They don't interrupt synchronous JavaScript.

They receive priority only after the current work finishes.

---

# A Common Interview Question

Predict the output.

```js
console.log(1);

setTimeout(() => console.log(2), 0);

Promise.resolve().then(() => console.log(3));

console.log(4);
```

Execution:

1. Print `1`
2. Register timer
3. Queue Promise callback
4. Print `4`
5. Empty Microtask Queue → `3`
6. Execute timer → `2`

Output:

```text
1
4
3
2
```

---

# React Connection

React frequently schedules work using Promises and other microtask-based mechanisms.

Understanding queue priority helps explain:

- why state updates may appear after the current event handler,
- why effects run after rendering,
- why Promise-based updates often occur before timer-based work.

While React has its own scheduler, it still operates on top of JavaScript's Event Loop.

---

# Key Takeaways

- JavaScript uses both Microtask and Macrotask queues.
- Promise callbacks are microtasks.
- Timer callbacks are macrotasks.
- The Event Loop always empties the Microtask Queue before executing the next Macrotask.
- Understanding queue priority is essential for predicting asynchronous execution.

---

> **Next Chapter:** *Promises — Solving Callback Hell with a Better Abstraction*
