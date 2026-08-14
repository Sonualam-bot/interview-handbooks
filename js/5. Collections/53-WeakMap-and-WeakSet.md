# Chapter 53 — WeakMap & WeakSet

> **Collections Handbook**

---

# What You'll Learn

- Why `WeakMap` and `WeakSet` Exist
- What Makes Them "Weak"
- Weak References
- `WeakMap`
- `WeakSet`
- Relationship with Garbage Collection
- Limitations
- Interview Questions

---

# Introduction

In the previous chapters, we learned about `Map` and `Set`.

Both hold **strong references** to their keys and values. As long as those references exist, the JavaScript engine cannot reclaim the associated memory.

ES6 introduced **WeakMap** and **WeakSet** for scenarios where objects should not be kept alive solely because they are stored in a collection.

---

# Strong vs Weak References

Consider a `Map`.

```js
const map = new Map();

let user = { name: "Sonu" };

map.set(user, "Admin");
```

Even if we later do:

```js
user = null;
```

the object is still referenced by the `Map`, so it cannot be garbage collected.

A **WeakMap** behaves differently.

---

# What is a WeakMap?

A `WeakMap` is a collection of key-value pairs where:

- Keys **must be objects**
- Keys are held using **weak references**

```js
const weakMap = new WeakMap();
```

---

# Adding Entries

```js
const user = { id: 1 };

const permissions = new WeakMap();

permissions.set(user, "Admin");

console.log(
  permissions.get(user)
);
```

Output:

```text
Admin
```

---

# Why Only Objects?

```js
const wm = new WeakMap();

wm.set("name", "Sonu");
```

Output:

```text
TypeError
```

Primitive values cannot be weakly referenced.

---

# Garbage Collection

```js
let user = {
  id: 1
};

const cache = new WeakMap();

cache.set(user, {
  profile: "Loaded"
});

user = null;
```

Once no strong references to the original object remain, the JavaScript engine is free to remove the entry from the `WeakMap`.

The exact timing is implementation-dependent.

---

# WeakMap Limitations

Unlike `Map`, a `WeakMap`:

- Cannot be iterated
- Has no `size`
- Has no `keys()`
- Has no `values()`
- Has no `entries()`

This is because entries may disappear at any time due to garbage collection.

---

# What is a WeakSet?

A `WeakSet` is similar to a `Set`, but:

- Only stores objects
- Stores them using weak references

```js
const weakSet = new WeakSet();

const user = {};

weakSet.add(user);

console.log(
  weakSet.has(user)
);
```

---

# WeakSet Limitations

`WeakSet` also cannot be iterated.

It has:

- `add()`
- `has()`
- `delete()`

but no `size` or iteration methods.

---

# Map vs WeakMap

| Feature | Map | WeakMap |
|---------|-----|---------|
| Keys | Any value | Objects only |
| Iterable | ✅ | ❌ |
| `size` | ✅ | ❌ |
| Prevents GC | Yes | No |

---

# Set vs WeakSet

| Feature | Set | WeakSet |
|---------|-----|----------|
| Values | Any value | Objects only |
| Iterable | ✅ | ❌ |
| `size` | ✅ | ❌ |
| Weak References | ❌ | ✅ |

---

# When to Use WeakMap

Common use cases:

- Private metadata for objects
- Caching object-related data
- Associating information with DOM nodes
- Preventing memory leaks

---

# Common Mistakes

### Using Primitive Keys

```js
weakMap.set(1, "A");
```

This throws a `TypeError`.

---

### Expecting Iteration

```js
for (const item of weakMap) {}
```

Not supported.

---

# Interview Questions

### Why was `WeakMap` introduced?

To associate data with objects without preventing garbage collection.

---

### Why can't `WeakMap` be iterated?

Entries may disappear at any time due to garbage collection.

---

### Why must keys be objects?

Only objects can be weakly referenced.

---

# Key Takeaways

- `WeakMap` stores object keys using weak references.
- `WeakSet` stores object values using weak references.
- Neither collection supports iteration or a `size` property.
- They are useful for caches, metadata, and avoiding memory leaks.
- Understanding them requires knowledge of JavaScript garbage collection.

---

# Next Chapter

**Chapter 54 — Iterables & Iterators**
