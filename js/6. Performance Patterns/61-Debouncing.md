# Chapter 61 — Debouncing

> **Performance Patterns Handbook**

---

# What You'll Learn

- Why Debouncing Exists
- How Debouncing Works
- Building a Debounce Function
- Practical Use Cases
- Debouncing vs Normal Event Handling
- Common Mistakes
- Interview Questions

---

# Introduction

Modern web applications respond to events such as:

- Typing
- Window resizing
- Searching
- Mouse movement

These events can fire dozens or even hundreds of times per second.

Executing expensive logic for every event can slow down an application.

**Debouncing** solves this problem.

---

# What is Debouncing?

Debouncing delays the execution of a function until a specified amount of time has passed **without another event occurring**.

Every new event resets the timer.

Conceptually:

```text
Typing Events

| | | | | |

<-- timer resets -->

........500ms........

Execute Once
```

The function executes only after the user stops triggering events.

---

# Without Debouncing

```js
input.addEventListener("input", () => {
  console.log("Searching...");
});
```

Every keystroke triggers a search.

Typing "JavaScript" may produce ten API calls.

---

# With Debouncing

```js
function debounce(fn, delay) {
  let timer;

  return function (...args) {
    clearTimeout(timer);

    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}
```

Usage:

```js
const search = debounce(() => {
  console.log("Searching...");
}, 500);

input.addEventListener("input", search);
```

Now the search executes only after the user stops typing for 500ms.

---

# How Debouncing Works

```text
Event

↓

Cancel Previous Timer

↓

Start New Timer

↓

No New Event?

├── No → Reset Timer
└── Yes
      │
      ▼
 Execute Function
```

---

# Practical Use Cases

Debouncing is commonly used for:

- Search boxes
- Auto-save functionality
- Form validation
- Window resize calculations
- API requests while typing

---

# Debouncing vs Normal Event Handling

Without debouncing:

```text
Key Presses

A B C D E

↓

API API API API API
```

With debouncing:

```text
Key Presses

A B C D E

↓

One API Request
```

---

# Common Mistakes

## Forgetting to Preserve `this`

Use:

```js
fn.apply(this, args);
```

instead of:

```js
fn(args);
```

This preserves the original context and arguments.

---

## Debouncing Everything

Not every event should be debounced.

For example, animations and scroll position updates often require throttling instead.

---

# Interview Questions

### What is debouncing?

Debouncing delays function execution until events stop occurring for a specified duration.

---

### Why is debouncing useful?

It prevents unnecessary repeated work and improves performance.

---

### Where is debouncing commonly used?

Search inputs, form validation, auto-save, and resize events.

---

# Key Takeaways

- Debouncing waits until user activity stops.
- Every new event resets the timer.
- Debouncing reduces unnecessary API calls and expensive computations.
- It is one of the most common frontend interview topics.

---

# Next Chapter

**Chapter 62 — Throttling**
