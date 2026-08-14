# Chapter 24 — Callbacks

> *"Sometimes writing a function isn't enough. Sometimes you need to give someone else the power to decide **when** your function should run."*

---

# A Mystery

Consider this code.

```js
console.log("Start");

setTimeout(() => {
    console.log("Done");
}, 1000);

console.log("End");
```

Output:

```text
Start
End
Done
```

The callback was written before `"End"`.

So why wasn't it executed first?

Who decided to delay it?

---

# Becoming the JavaScript Engine

Imagine you're executing:

```js
setTimeout(callback, 1000);
```

You recognize `setTimeout()` as a function provided by the runtime.

It asks you for another function.

You don't execute that function immediately.

Instead, you hand it to the runtime with one instruction:

> "Run this function after approximately one second."

Execution of the current program continues uninterrupted.

The callback waits until someone else decides it's time.

---

# What Is a Callback?

A **callback** is a function passed to another function so it can be executed later, at the appropriate time.

The caller provides **what** should happen.

The receiving function decides **when** it should happen.

---

# Think of Ordering Food

Imagine ordering food at a restaurant.

You don't stand in the kitchen waiting.

Instead, you leave your phone number.

When your meal is ready, the restaurant calls you back.

You provided the action.

The restaurant decided the timing.

A callback follows the same idea.

---

# A Simple Callback

```js
function greet() {
    console.log("Hello!");
}

function execute(task) {
    task();
}

execute(greet);
```

`execute()` doesn't know what `greet()` does.

It simply decides when to invoke it.

---

# Synchronous Callbacks

Not every callback is asynchronous.

```js
const numbers = [1, 2, 3];

numbers.forEach(number => {
    console.log(number);
});
```

The callback runs immediately for each array element.

The array controls the iteration.

Your callback supplies the behavior.

---

# Asynchronous Callbacks

Other callbacks run much later.

```js
setTimeout(() => {
    console.log("One second passed");
}, 1000);
```

The callback waits until the timer expires.

Your code continues executing in the meantime.

This distinction becomes important when learning the Event Loop.

---

# Why Callbacks Exist

Imagine if every API hardcoded its behavior.

`setTimeout()` could only print messages.

`map()` could only double numbers.

`addEventListener()` could only respond one specific way.

Callbacks make APIs flexible.

The API controls the process.

You control the behavior.

---

# Callback Hell

As applications grew larger, developers often nested callbacks.

```js
login(() => {
    fetchProfile(() => {
        fetchPosts(() => {
            // ...
        });
    });
});
```

Deep nesting became difficult to read and maintain.

This problem became known as **callback hell**.

Promises and `async`/`await` were introduced to provide a cleaner approach while still relying on callbacks under the hood.

---

# React Connection

React applications use callbacks constantly.

Examples include:

```jsx
<button onClick={handleClick} />
```

```jsx
<input onChange={handleChange} />
```

```js
setState(prev => prev + 1);
```

In every case, React decides **when** your callback executes.

You only define **what** should happen.

---

# Key Takeaways

- A callback is a function passed to another function.
- The receiving function decides when the callback executes.
- Callbacks may be synchronous or asynchronous.
- Many JavaScript APIs rely on callbacks.
- React event handlers and updater functions are callback-based.

---

> **Next Chapter:** *The Event Loop — How JavaScript Handles Asynchronous Work Without Multiple Threads*
