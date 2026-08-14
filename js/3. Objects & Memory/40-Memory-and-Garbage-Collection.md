# Chapter 40 — Memory & Garbage Collection

> **Objects & Memory Handbook**

---

# What You'll Learn

- Why Garbage Collection Exists
- Reachability
- How JavaScript Frees Memory
- Common Causes of Memory Leaks
- Garbage Collection Algorithms
- Weak References
- Best Practices
- Interview Questions

---

# Introduction

In languages like C and C++, developers manually allocate and free memory.

```c
malloc(...);
free(...);
```

Forgetting to free memory causes leaks, while freeing memory incorrectly can crash a program.

JavaScript takes a different approach.

The JavaScript engine automatically manages memory using a **Garbage Collector (GC)**.

---

# Why Garbage Collection?

Every program continuously creates values.

```js
let user = {
  name: "Sonu"
};

let numbers = [1, 2, 3];
```

If unused memory were never released, applications would eventually consume all available memory.

The Garbage Collector automatically reclaims memory that is no longer needed.

---

# The Memory Lifecycle

Every value goes through three stages.

1. Allocation
2. Usage
3. Release

```text
Allocation
     │
     ▼
  Program Uses
     │
     ▼
No Longer Reachable
     │
     ▼
Garbage Collector Reclaims Memory
```

---

# Reachability

JavaScript decides whether memory can be reclaimed using the concept of **reachability**.

A value is **reachable** if it can still be accessed.

```js
let user = {
  name: "Sonu"
};
```

The object is reachable because `user` points to it.

---

# Becoming Unreachable

```js
let user = {
  name: "Sonu"
};

user = null;
```

Conceptually:

```text
Before

user ─────► { name: "Sonu" }

After

user ─────► null

{ name: "Sonu" }   ← unreachable
```

Once nothing references the object, it becomes eligible for garbage collection.

---

# Garbage Collection is Automatic

Developers do **not** manually free memory.

The JavaScript engine decides:

- When to run the garbage collector
- Which objects are unreachable
- When memory should be reclaimed

This behavior varies between engines such as V8, SpiderMonkey, and JavaScriptCore.

---

# Mark-and-Sweep

Modern JavaScript engines primarily use the **Mark-and-Sweep** algorithm.

Conceptually:

1. Start from root objects (global variables, execution contexts, etc.).
2. Mark every reachable object.
3. Sweep away anything that was not marked.

```text
Roots
 │
 ▼
Reachable Objects ✔

Unreachable Objects ✖
        │
        ▼
Removed
```

---

# Common Causes of Memory Leaks

Garbage collection is automatic, but memory leaks can still happen if objects remain reachable unintentionally.

Examples include:

- Global variables
- Uncleared timers
- Event listeners that are never removed
- Closures retaining large objects
- Growing caches

Example:

```js
const users = [];

setInterval(() => {
  users.push({ time: Date.now() });
}, 1000);
```

If `users` keeps growing forever, memory usage also keeps growing.

---

# Weak References

Sometimes we don't want an object to prevent garbage collection.

JavaScript provides:

- `WeakMap`
- `WeakSet`

These hold **weak references**, allowing objects to be collected when no other references exist.

We'll study these advanced structures later.

---

# Best Practices

- Remove unused event listeners.
- Clear timers with `clearTimeout()` and `clearInterval()`.
- Avoid unnecessary global variables.
- Release large caches when no longer needed.
- Prefer local scope whenever possible.

---

# Interview Questions

### What is Garbage Collection?

An automatic process that reclaims memory occupied by objects that are no longer reachable.

---

### What does "reachable" mean?

An object is reachable if it can still be accessed directly or indirectly from a root object.

---

### Can JavaScript developers manually free memory?

No.

Memory management is handled automatically by the JavaScript engine.

---

### Which algorithm is commonly used?

Modern engines primarily use **Mark-and-Sweep**.

---

# Key Takeaways

- JavaScript automatically manages memory.
- Reachability determines whether an object can be collected.
- Modern engines primarily use the Mark-and-Sweep algorithm.
- Memory leaks occur when objects remain reachable unintentionally.
- Understanding garbage collection is essential for writing efficient JavaScript applications.

---

# Handbook Complete ✅

You have completed the **Objects & Memory Handbook**.

Next:

**JavaScript OOP Handbook**
- Chapter 41 — Constructor Functions
