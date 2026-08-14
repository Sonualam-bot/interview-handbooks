# Chapter 62 — Throttling

> **Performance Patterns Handbook**

---

# What You'll Learn

- Why Throttling Exists
- How Throttling Works
- Building a Throttle Function
- Practical Use Cases
- Throttling vs Debouncing
- Common Mistakes
- Interview Questions

---

# Introduction

Some browser events fire continuously while the user interacts with the page.

Examples include:

- Scrolling
- Mouse movement
- Window resizing
- Touch events

Running expensive logic for every event can hurt performance.

**Throttling** limits how often a function can execute.

---

# What is Throttling?

Throttling ensures a function executes **at most once** during a specified time interval.

Unlike debouncing, throttling does **not** wait for events to stop.

Conceptually:

```text
Events

| | | | | | | | |

Throttle Interval = 500ms

Execute ---- Execute ---- Execute
```

Even if many events occur, the function runs at a controlled rate.

---

# Without Throttling

```js
window.addEventListener("scroll", () => {
  console.log("Scroll event");
});
```

Every scroll event triggers the callback.

---

# Building a Throttle Function

```js
function throttle(fn, delay) {
  let waiting = false;

  return function (...args) {
    if (waiting) return;

    fn.apply(this, args);

    waiting = true;

    setTimeout(() => {
      waiting = false;
    }, delay);
  };
}
```

Usage:

```js
const handleScroll = throttle(() => {
  console.log("Updating UI...");
}, 500);

window.addEventListener("scroll", handleScroll);
```

---

# How Throttling Works

```text
Event

↓

Already Waiting?

├── Yes → Ignore Event
└── No
      │
      ▼
 Execute Function
      │
      ▼
 Wait Delay
      │
      ▼
 Accept Next Event
```

---

# Practical Use Cases

Throttling is commonly used for:

- Scroll listeners
- Mouse movement
- Infinite scrolling
- Progress indicators
- Resize calculations
- Drag-and-drop interactions

---

# Throttling vs Debouncing

| Throttling | Debouncing |
|------------|------------|
| Executes at fixed intervals | Executes after events stop |
| Good for continuous updates | Good for final actions |
| Ignores extra events during the interval | Resets the timer on every event |

Example:

Scroll tracking → **Throttle**

Search input → **Debounce**

---

# Common Mistakes

## Using Debounce Instead of Throttle

For continuous UI updates like scrolling, debouncing may delay updates too much.

Throttling provides smoother behavior.

---

## Forgetting to Preserve `this`

Use:

```js
fn.apply(this, args);
```

instead of directly calling `fn()` to preserve the original context and arguments.

---

# Interview Questions

### What is throttling?

Throttling limits a function so it executes at most once within a specified time interval.

---

### What is the difference between throttling and debouncing?

- Throttling executes periodically.
- Debouncing executes only after activity has stopped.

---

### Where is throttling commonly used?

Scroll events, resize events, mouse movement, and other high-frequency UI events.

---

# Key Takeaways

- Throttling controls how frequently a function executes.
- It is ideal for high-frequency events.
- A throttle implementation typically tracks whether execution is currently allowed.
- Throttling improves UI responsiveness and application performance.
- Understanding the difference between throttling and debouncing is a common interview topic.

---

# Next Chapter

**Chapter 63 — Error Handling**
