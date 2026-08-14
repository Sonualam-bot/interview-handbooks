# Chapter 58 — Currying

> **Performance Patterns Handbook**

---

# What You'll Learn

- What Currying Is
- Why Currying Exists
- How Currying Works
- Currying vs Normal Functions
- Generic Curry Function
- Practical Use Cases
- Common Mistakes
- Interview Questions

---

# Introduction

Currying is a functional programming technique that transforms a function
accepting multiple arguments into a sequence of functions that each accept
one argument.

Instead of:

```js
add(1, 2, 3);
```

we can write:

```js
add(1)(2)(3);
```

Currying is a common JavaScript interview topic because it combines closures,
higher-order functions, and function composition.

---

# Normal Function

```js
function add(a, b, c) {
  return a + b + c;
}

console.log(add(1, 2, 3));
```

Output:

```text
6
```

---

# Curried Function

```js
function add(a) {
  return function (b) {
    return function (c) {
      return a + b + c;
    };
  };
}

console.log(add(1)(2)(3));
```

Output:

```text
6
```

Each returned function remembers the previous arguments using closures.

---

# Why Curry Functions?

Currying allows you to create specialized functions from generic ones.

```js
const multiply = a => b => a * b;

const double = multiply(2);
const triple = multiply(3);

console.log(double(5));
console.log(triple(5));
```

Output:

```text
10
15
```

---

# How Currying Works

Conceptually:

```text
add(1)(2)(3)

↓

a = 1

↓

b = 2

↓

c = 3

↓

return 6
```

Every function remembers the arguments supplied before it.

---

# Generic Curry Function

Instead of manually writing nested functions every time, we can create a reusable helper.

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn(...args);
    }

    return (...nextArgs) =>
      curried(...args, ...nextArgs);
  };
}
```

Usage:

```js
function sum(a, b, c) {
  return a + b + c;
}

const curriedSum = curry(sum);

console.log(curriedSum(1)(2)(3));
console.log(curriedSum(1, 2)(3));
console.log(curriedSum(1)(2, 3));
```

Output:

```text
6
6
6
```

---

# Practical Use Cases

Currying is useful for:

- Function factories
- Configuration APIs
- Event handlers
- Functional programming libraries
- Creating reusable utility functions

---

# Currying vs Partial Application

Currying transforms the function so that it accepts one argument at a time.

Partial application fixes some arguments of an existing function.

We'll study partial application in the next chapter.

---

# Common Mistakes

## Confusing Currying with Nested Functions

Not every nested function is curried.

A curried function represents a transformation of a multi-parameter function.

---

## Assuming JavaScript Supports Currying Automatically

JavaScript does not curry functions by default.

Currying is implemented manually or with helper utilities.

---

# Interview Questions

### What is currying?

Currying transforms a function with multiple parameters into a sequence of functions that each accept a single argument.

---

### Why is currying useful?

It improves code reuse, composition, and enables specialized functions.

---

### Does JavaScript support currying natively?

No.

Currying is implemented using closures or helper functions.

---

# Key Takeaways

- Currying converts multi-parameter functions into chains of single-parameter functions.
- Closures preserve previously supplied arguments.
- Curried functions enable reusable and composable code.
- Generic curry helpers are common interview questions.
- Currying is different from partial application.

---

# Next Chapter

**Chapter 59 — Partial Application**
