# Chapter 63 — Error Handling

> **Performance Patterns Handbook**

---

# What You'll Learn

- Why Error Handling Matters
- Types of Errors
- `try...catch`
- `finally`
- `throw`
- Creating Custom Errors
- Best Practices
- Common Interview Questions

---

# Introduction

Errors are inevitable in software.

A network request may fail, user input may be invalid, or an unexpected value may be received.

JavaScript provides structured error handling so applications can recover gracefully instead of crashing.

---

# Types of Errors

Common built-in error types include:

- `SyntaxError`
- `ReferenceError`
- `TypeError`
- `RangeError`
- `URIError`

Example:

```js
const obj = null;

console.log(obj.name);
```

Output:

```text
TypeError: Cannot read properties of null
```

---

# try...catch

Use `try...catch` to handle runtime errors.

```js
try {
  const user = null;
  console.log(user.name);
} catch (error) {
  console.log("Something went wrong.");
  console.log(error.message);
}
```

Output:

```text
Something went wrong.
Cannot read properties of null
```

Execution continues after the `catch` block.

---

# finally

The `finally` block always executes, whether an error occurs or not.

```js
try {
  console.log("Opening connection");
} finally {
  console.log("Closing connection");
}
```

Typical use cases:

- Closing files
- Releasing resources
- Stopping loaders
- Cleaning up state

---

# throw

Use `throw` to create your own errors.

```js
function divide(a, b) {
  if (b === 0) {
    throw new Error("Division by zero");
  }

  return a / b;
}
```

Handle it with:

```js
try {
  divide(10, 0);
} catch (error) {
  console.log(error.message);
}
```

---

# The Error Object

The built-in `Error` object contains useful information.

```js
try {
  throw new Error("Something failed");
} catch (error) {
  console.log(error.name);
  console.log(error.message);
}
```

Output:

```text
Error
Something failed
```

---

# Custom Errors

Create custom error classes for domain-specific failures.

```js
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError";
  }
}

throw new ValidationError("Invalid email");
```

This makes errors easier to identify and handle.

---

# Best Practices

- Catch only errors you can handle.
- Throw meaningful error messages.
- Avoid empty `catch` blocks.
- Use `finally` for cleanup logic.
- Prefer custom error classes for large applications.

---

# Common Mistakes

## Swallowing Errors

```js
try {
  riskyOperation();
} catch (error) {}
```

Ignoring errors makes debugging difficult.

---

## Using `throw` with Strings

Avoid:

```js
throw "Something went wrong";
```

Prefer:

```js
throw new Error("Something went wrong");
```

---

# Interview Questions

### What is the purpose of `try...catch`?

It allows runtime errors to be caught and handled without terminating the application.

---

### When does `finally` execute?

Always—whether an error occurs or not.

---

### Why create custom error classes?

They make different categories of errors easier to identify and handle.

---

### What is the difference between `throw` and `catch`?

- `throw` creates or propagates an error.
- `catch` receives and handles the error.

---

# Key Takeaways

- `try...catch` handles runtime errors.
- `finally` always executes and is useful for cleanup.
- `throw` creates custom errors.
- The `Error` object provides `name` and `message`.
- Good error handling improves reliability and debugging.

---

# Next Chapter

**Chapter 64 — Regular Expressions (Interview Edition)**
