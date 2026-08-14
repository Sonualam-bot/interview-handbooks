# Chapter 59 — Partial Application

> **Performance Patterns Handbook**

---

# What You'll Learn

- What Partial Application Is
- Why It Exists
- How Partial Application Works
- Partial Application vs Currying
- Using `bind()`
- Practical Use Cases
- Common Mistakes
- Interview Questions

---

# Introduction

Partial application is a technique where some arguments of a function are
pre-filled, producing a new function that expects the remaining arguments.

Unlike currying, the original function is **not** transformed into a chain of
single-argument functions.

---

# A Normal Function

```js
function multiply(a, b) {
  return a * b;
}

console.log(multiply(2, 5));
```

Output:

```text
10
```

---

# Partial Application

We can create a specialized function by fixing one argument.

```js
function multiply(a, b) {
  return a * b;
}

const double = multiply.bind(null, 2);

console.log(double(5));
```

Output:

```text
10
```

Here, `2` is permanently supplied as the first argument.

---

# How `bind()` Helps

`bind()` returns a new function.

```js
const triple = multiply.bind(null, 3);

console.log(triple(4));
```

Output:

```text
12
```

`null` is used because `multiply()` does not rely on `this`.

---

# Partial Application Without `bind()`

You can also implement it using closures.

```js
function partialMultiply(a) {
  return function (b) {
    return multiply(a, b);
  };
}

const double = partialMultiply(2);

console.log(double(8));
```

Closures remember the arguments that were already supplied.

---

# Partial Application vs Currying

| Currying | Partial Application |
|----------|---------------------|
| Converts a function into nested single-argument functions | Fixes some arguments of an existing function |
| Changes the function's structure | Reuses the original function |
| Often implemented with closures | Often implemented using `bind()` or closures |

Example:

Currying:

```js
add(1)(2)(3);
```

Partial application:

```js
const addOne = add.bind(null, 1);

addOne(2, 3);
```

---

# Practical Use Cases

Partial application is useful for:

- Event handlers
- Utility functions
- API wrappers
- Logging
- Configurable helper functions

Example:

```js
function log(level, message) {
  console.log(`[${level}] ${message}`);
}

const info = log.bind(null, "INFO");

info("Server started");
```

Output:

```text
[INFO] Server started
```

---

# Common Mistakes

## Confusing Partial Application with Currying

These techniques are related but different.

Partial application fixes arguments.

Currying changes how arguments are supplied.

---

## Forgetting that `bind()` Returns a New Function

```js
const fn = multiply.bind(null, 2);
```

The original function remains unchanged.

---

# Interview Questions

### What is partial application?

A technique that creates a new function by fixing some arguments of an existing function.

---

### How is partial application different from currying?

Currying transforms the function into nested functions.

Partial application simply pre-fills some arguments.

---

### Can `bind()` be used for partial application?

Yes.

`bind()` can fix both `this` and function arguments.

---

# Key Takeaways

- Partial application creates specialized functions by fixing some arguments.
- `bind()` is a common way to implement it.
- Closures can also implement partial application.
- Partial application and currying solve similar problems but are different techniques.
- This pattern appears in functional programming and JavaScript interviews.

---

# Next Chapter

**Chapter 60 — Memoization**
