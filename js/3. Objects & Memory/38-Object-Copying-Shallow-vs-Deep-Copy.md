# Chapter 38 — Object Copying (Shallow vs Deep Copy)

> **Objects & Memory Handbook**

---

# What You'll Learn

- Why Object Copying Matters
- Assignment vs Copying
- Shallow Copy
- Deep Copy
- Common Copying Techniques
- `structuredClone()`
- Common Mistakes
- Interview Questions

---

# Introduction

Objects are **reference types**.

This means copying an object is not as straightforward as copying a number or a string.

Consider this example:

```js
const user1 = {
  name: "Sonu"
};

const user2 = user1;

user2.name = "Rahul";

console.log(user1.name);
```

Output:

```text
Rahul
```

Why?

Because **no new object was created**.

Both variables point to the same object in memory.

---

# Assignment is NOT Copying

```js
const original = {
  name: "Sonu"
};

const copy = original;
```

Conceptually:

```text
Stack

original ─────┐
              │
copy ─────────┘
              │
              ▼

Heap

{
  name: "Sonu"
}
```

Only the reference is copied.

---

# What is a Shallow Copy?

A shallow copy creates a **new top-level object**, but nested objects are still shared.

```js
const original = {
  name: "Sonu"
};

const copy = {
  ...original
};

copy.name = "Rahul";

console.log(original.name);
```

Output:

```text
Sonu
```

The top-level property is independent.

---

# The Problem with Nested Objects

```js
const original = {
  address: {
    city: "Bangalore"
  }
};

const copy = {
  ...original
};

copy.address.city = "Delhi";

console.log(original.address.city);
```

Output:

```text
Delhi
```

Why?

Because only the outer object was copied.

The nested object is still shared.

---

# Ways to Create a Shallow Copy

## Spread Operator

```js
const copy = {
  ...original
};
```

---

## Object.assign()

```js
const copy = Object.assign({}, original);
```

Both approaches perform a shallow copy.

---

# What is a Deep Copy?

A deep copy duplicates **every nested object**.

After a deep copy:

```js
copy.address.city = "Delhi";
```

does **not** affect the original.

Conceptually:

```text
original ──► Object A

copy ──────► Object B
```

Each object has its own nested objects.

---

# structuredClone()

Modern JavaScript provides:

```js
const copy = structuredClone(original);
```

This creates a deep copy for supported data types.

---

# JSON Technique

A common older approach is:

```js
const copy =
  JSON.parse(JSON.stringify(original));
```

Limitations:

- Loses functions
- Loses `Date`
- Loses `Map`
- Loses `Set`
- Fails for circular references

It should only be used for simple data.

---

# Choosing the Right Technique

| Technique | Copy Type |
|-----------|-----------|
| Assignment (`=`) | No copy |
| Spread (`...`) | Shallow |
| `Object.assign()` | Shallow |
| `structuredClone()` | Deep |

---

# Common Mistakes

### Assuming Spread Creates a Deep Copy

```js
const copy = {
  ...original
};
```

Only the first level is copied.

Nested objects remain shared.

---

### Using JSON for Complex Objects

`JSON.stringify()` cannot preserve many JavaScript data types.

Prefer `structuredClone()` when available.

---

# Interview Questions

### What is the difference between a shallow copy and a deep copy?

A shallow copy duplicates only the first level.

A deep copy duplicates the entire object graph.

---

### Does the spread operator create a deep copy?

No.

It creates a shallow copy.

---

### When should `structuredClone()` be used?

When an independent deep copy of an object is required.

---

# Key Takeaways

- Assignment copies references, not objects.
- Spread and `Object.assign()` create shallow copies.
- Nested objects remain shared after a shallow copy.
- `structuredClone()` performs a deep copy for supported types.
- Understanding object copying is a very common JavaScript interview topic.

---

# Next Chapter

**Chapter 39 — Object Equality**
