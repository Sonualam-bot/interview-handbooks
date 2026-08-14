# Chapter 60 — Memoization

> **Performance Patterns Handbook**

---

# What You'll Learn

- What Memoization Is
- Why Memoization Exists
- Cache vs Memoization
- Building a Memoize Function
- Practical Use Cases
- Limitations
- Interview Questions

---

# Introduction

Some functions perform expensive calculations.

```js
function square(n) {
  console.log("Computing...");
  return n * n;
}

square(5);
square(5);
```

Output:

```text
Computing...
Computing...
```

The same result is calculated twice.

**Memoization** avoids repeated work by remembering previous results.

---

# What is Memoization?

Memoization is an optimization technique where the result of a function call is cached.

If the function is called again with the same inputs, the cached value is returned instead of recomputing it.

---

# A Simple Cache

```js
const cache = {};

function square(n) {
  if (cache[n] !== undefined) {
    return cache[n];
  }

  console.log("Computing...");

  const result = n * n;
  cache[n] = result;

  return result;
}

console.log(square(5));
console.log(square(5));
```

Output:

```text
Computing...
25
25
```

---

# Generic Memoize Function

```js
function memoize(fn) {
  const cache = new Map();

  return function (...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      return cache.get(key);
    }

    const result = fn(...args);

    cache.set(key, result);

    return result;
  };
}
```

Usage:

```js
function add(a, b) {
  console.log("Calculating...");
  return a + b;
}

const memoizedAdd = memoize(add);

memoizedAdd(2, 3);
memoizedAdd(2, 3);
```

The second call returns immediately from the cache.

---

# How Memoization Works

```text
Function Call

↓

Create Cache Key

↓

Key Exists?

├── Yes → Return Cached Value
└── No
      │
      ▼
 Execute Function
      │
      ▼
 Store Result
      │
      ▼
 Return Result
```

---

# Memoization vs Caching

| Memoization | General Caching |
|-------------|-----------------|
| Stores function results | Stores arbitrary data |
| Keys come from function arguments | Keys chosen by developer |
| Usually local to one function | May be application-wide |

---

# Practical Use Cases

Memoization is useful for:

- Expensive mathematical calculations
- Recursive algorithms
- Dynamic Programming
- Data transformation
- React selectors
- Derived state calculations

---

# Limitations

Memoization works best with **pure functions**.

Avoid memoizing functions that:

- Depend on external state
- Produce side effects
- Return different results for the same inputs

Also remember:

- Caches consume memory.
- Large caches may need eviction strategies.

---

# Common Mistakes

## Memoizing Impure Functions

```js
function random() {
  return Math.random();
}
```

Memoizing this function defeats its purpose.

---

## Forgetting Cache Growth

A cache that grows forever may increase memory usage.

---

# Interview Questions

### What is memoization?

Memoization stores the results of previous function calls and reuses them for identical inputs.

---

### When should memoization be used?

When a pure function is expensive and frequently called with the same arguments.

---

### Is memoization the same as caching?

Memoization is a specialized form of caching focused on function results.

---

# Key Takeaways

- Memoization improves performance by avoiding repeated computation.
- Cached values are returned for repeated inputs.
- `Map` is commonly used to implement memoization.
- Memoization is most effective for pure functions.
- It is a common optimization technique discussed in JavaScript interviews.

---

# Next Chapter

**Chapter 61 — Debouncing**
