# Chapter 39 — Object Equality

> **Objects & Memory Handbook**

---

# What You'll Learn

- Equality in JavaScript
- `==` vs `===`
- Object Equality
- Reference Equality
- Value Equality
- Comparing Objects
- Common Mistakes
- Interview Questions

---

# Introduction

One of the most confusing JavaScript behaviors is object comparison.

Consider:

```js
const obj1 = { name: "Sonu" };
const obj2 = { name: "Sonu" };

console.log(obj1 === obj2);
```

Output:

```text
false
```

Even though both objects contain the same data, JavaScript considers them different.

Understanding **why** is a common interview question.

---

# Equality Operators

JavaScript provides two commonly used equality operators.

| Operator | Meaning |
|----------|---------|
| `==` | Loose equality |
| `===` | Strict equality |

For objects, both compare **references**, not contents.

---

# Primitive Equality

Primitive values are compared by their actual value.

```js
console.log(10 === 10);
console.log("Hello" === "Hello");
console.log(true === true);
```

Output:

```text
true
true
true
```

---

# Object Equality

Objects are compared by their identity (reference).

```js
const a = { x: 1 };
const b = { x: 1 };

console.log(a === b);
```

Output:

```text
false
```

`a` and `b` point to different objects in memory.

---

# Reference Equality

```js
const user = {
  name: "Sonu"
};

const copy = user;

console.log(user === copy);
```

Output:

```text
true
```

Both variables reference the same object.

Conceptually:

```text
Stack

user ────┐
         │
copy ────┘
         │
         ▼

Heap

{
  name: "Sonu"
}
```

---

# Nested Objects

```js
const obj1 = {
  address: {
    city: "Bangalore"
  }
};

const obj2 = {
  address: {
    city: "Bangalore"
  }
};

console.log(obj1 === obj2);
```

Output:

```text
false
```

The outer objects are different references.

---

# Comparing Object Contents

JavaScript has no built-in deep equality operator.

A simple (but limited) approach is:

```js
const equal =
  JSON.stringify(obj1) === JSON.stringify(obj2);
```

Limitations:

- Property order matters
- Doesn't handle functions
- Doesn't handle `Map`, `Set`, `Date`
- Doesn't support circular references

For production code, use a dedicated deep-equality utility.

---

# Object.is()

`Object.is()` is similar to `===` but differs for a few edge cases.

```js
Object.is(NaN, NaN);   // true
Object.is(+0, -0);     // false

NaN === NaN;           // false
+0 === -0;             // true
```

For ordinary objects:

```js
Object.is(obj1, obj2);
```

still compares references.

---

# Common Mistakes

### Assuming identical objects are equal

```js
{} === {}
```

Output:

```text
false
```

Each object literal creates a new object.

---

### Using `==` for Objects

```js
obj1 == obj2
```

This also compares references.

---

# Interview Questions

### Why does `{}` === `{}` return `false`?

Because each object literal creates a different object in memory.

---

### How are objects compared in JavaScript?

By reference, not by their contents.

---

### Can JavaScript compare two objects deeply?

Not using `===`.

A custom deep comparison or library is required.

---

# Key Takeaways

- Primitive values are compared by value.
- Objects are compared by reference.
- Two identical object literals are never equal unless they reference the same object.
- `Object.is()` behaves like `===` with a few edge-case differences.
- Deep equality requires additional logic.

---

# Next Chapter

**Chapter 40 — Memory & Garbage Collection**
