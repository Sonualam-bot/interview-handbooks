# Chapter 25 — The Event Loop

> *"JavaScript executes one line at a time on a single thread. Yet it can download files, respond to clicks, wait for timers, and update the UI without freezing. How?"*

---

# A Mystery

Consider this code.

```js
console.log("A");

setTimeout(() => {
    console.log("B");
}, 0);

console.log("C");
```

Many beginners expect:

```text
A
B
C
```

But the actual output is:

```text
A
C
B
```

The timer was set to **0 milliseconds**.

So why didn't it execute immediately?

---

# Becoming the JavaScript Engine

Imagine you're executing this program.

You begin with:

```js
console.log("A");
```

The Call Stack executes it.

Next you encounter:

```js
setTimeout(callback, 0);
```

You don't pause execution.

Instead, you ask the runtime to start a timer.

The callback is **registered**, not executed.

You immediately continue to:

```js
console.log("C");
```

Only after the Call Stack becomes empty can the callback be considered for execution.

---

# JavaScript Is Single-Threaded

JavaScript executes one piece of JavaScript code at a time.

There is only one Call Stack.

Only one function can actively execute at any moment.

This makes execution predictable, but it raises a question:

> How can JavaScript appear to do many things at once?

The answer is that **the JavaScript engine is not working alone**.

---

# The Runtime

The browser (or Node.js) provides additional capabilities such as:

- Timers
- Network requests
- DOM events
- File operations (Node.js)

These features run outside the JavaScript engine.

When they finish, they notify JavaScript that work is ready.

---

# The Callback Queue

Completed asynchronous tasks place their callbacks into the **Callback Queue**.

Think of it as a waiting room.

Callbacks are ready to execute, but they must wait until the Call Stack is empty.

---

# The Event Loop

The Event Loop repeatedly asks one question:

> "Is the Call Stack empty?"

If the answer is yes, it moves the next callback from the queue onto the Call Stack.

Conceptually:

```text
Call Stack

↓

(empty?)

↓

Yes

↓

Move next callback from Queue

↓

Execute it
```

This cycle repeats continuously while your program is running.

---

# Why `setTimeout(..., 0)` Isn't Immediate

A delay of `0` milliseconds means:

> "The callback becomes eligible as soon as possible."

It does **not** mean:

> "Interrupt whatever JavaScript is currently doing."

Current synchronous work always finishes first.

---

# A Real-World Analogy

Imagine a chef preparing meals.

Customers keep placing new orders.

Finished dishes are lined up on a serving counter.

The waiter only serves the next dish after finishing the one already in hand.

The chef is like the runtime.

The serving counter is the callback queue.

The waiter is the Event Loop.

The waiter never serves two dishes simultaneously.

---

# Microtasks vs Macrotasks

Not every callback has equal priority.

Some operations, such as resolved Promises, enter the **Microtask Queue**.

Others, like `setTimeout()`, enter the **Macrotask (Callback) Queue**.

The Event Loop always processes:

1. Current synchronous code
2. All pending microtasks
3. The next macrotask

This is why Promise callbacks often execute before timer callbacks.

---

# React Connection

React relies heavily on the Event Loop.

Examples include:

- browser click events,
- asynchronous state updates,
- fetching API data,
- scheduling rendering work,
- batching updates.

Understanding the Event Loop helps explain why React sometimes updates the UI after the current function finishes rather than immediately.

---

# Key Takeaways

- JavaScript executes code on a single Call Stack.
- Timers and network operations are handled by the runtime.
- Completed asynchronous callbacks wait in queues.
- The Event Loop moves callbacks onto the Call Stack only when it is empty.
- Promise microtasks run before timer callbacks.

---

> **Next Chapter:** *Microtasks vs Macrotasks — Understanding JavaScript's Scheduling Priorities*
