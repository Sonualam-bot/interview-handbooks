# Chapter 51 — Map

> **Collections Handbook**

---

# What You'll Learn

- Why `Map` Was Introduced
- Creating a Map
- Adding, Reading and Removing Entries
- Iterating Over a Map
- `Map` vs `Object`
- Time Complexity
- Common Interview Questions

---

# Introduction

Before ES6, JavaScript developers primarily used plain objects as key-value stores.

```js
const userAges = {
  Sonu: 26,
  Rahul: 24
};
```

Objects work well in many cases, but they have limitations:

- Keys are usually strings or symbols.
- Objects inherit properties from `Object.prototype`.
- Determining the number of entries requires extra work.

ES6 introduced **Map** to solve these problems.

---

# What is a Map?

A `Map` is a collection of **key-value pairs** where **any JavaScript value** can be used as a key.

```js
const map = new Map();
```

Unlike objects, keys can be:

- Strings
- Numbers
- Booleans
- Objects
- Functions

---

# Adding Entries

Use `set()`.

```js
const users = new Map();

users.set("Sonu", 26);
users.set("Rahul", 24);

console.log(users);
```

`set()` returns the map, allowing method chaining.

```js
users
  .set("A", 1)
  .set("B", 2);
```

---

# Reading Values

Use `get()`.

```js
console.log(users.get("Sonu"));
```

Output:

```text
26
```

If the key does not exist:

```text
undefined
```

---

# Checking Keys

```js
console.log(users.has("Rahul"));
```

Output:

```text
true
```

---

# Removing Entries

```js
users.delete("Rahul");
```

Remove everything:

```js
users.clear();
```

---

# Size

Unlike objects, `Map` provides a built-in size property.

```js
const map = new Map();

map.set("A", 1);
map.set("B", 2);

console.log(map.size);
```

Output:

```text
2
```

---

# Object Keys

Objects can also be keys.

```js
const user = { id: 1 };

const permissions = new Map();

permissions.set(user, "Admin");

console.log(permissions.get(user));
```

Output:

```text
Admin
```

---

# Iterating Over a Map

```js
const scores = new Map([
  ["Sonu", 90],
  ["Rahul", 85]
]);

for (const [name, score] of scores) {
  console.log(name, score);
}
```

Useful methods:

```js
scores.keys();
scores.values();
scores.entries();
```

---

# Map vs Object

| Feature | Map | Object |
|--------|------|--------|
| Key Types | Any value | String / Symbol |
| Iteration | Built-in | Extra methods needed |
| Size | `size` | `Object.keys(obj).length` |
| Insertion Order | Preserved | Mostly preserved but object semantics differ |
| Designed for Key-Value Storage | ✅ | General-purpose object |

---

# Time Complexity

| Operation | Complexity |
|----------|------------|
| `set()` | O(1) average |
| `get()` | O(1) average |
| `has()` | O(1) average |
| `delete()` | O(1) average |

---

# When to Use Map

Use `Map` when:

- Keys are not strings.
- Frequent insertions and deletions occur.
- Key-value storage is the primary purpose.
- You need predictable iteration order.

Use plain objects for modeling structured entities like users, products, or API responses.

---

# Common Mistakes

### Using Dot Notation

```js
map.name = "Sonu";
```

This creates a normal object property, **not** a map entry.

Correct:

```js
map.set("name", "Sonu");
```

---

### Forgetting `get()`

```js
map["Sonu"];
```

This does not access map entries.

Use:

```js
map.get("Sonu");
```

---

# Interview Questions

### Why was `Map` introduced?

To provide a dedicated key-value collection supporting any type of key.

---

### When should you use `Map` instead of an object?

When keys are dynamic, non-string values, or when frequent insertions, deletions, and iteration are required.

---

### Can objects be used as keys in a `Map`?

Yes.

Each object reference is treated as a unique key.

---

# Key Takeaways

- `Map` is a dedicated key-value collection.
- Keys can be any JavaScript value.
- `set()`, `get()`, `has()`, and `delete()` are the primary operations.
- `Map` preserves insertion order and provides a built-in `size`.
- `Map` is often preferred over objects for dynamic key-value storage.

---

# Next Chapter

**Chapter 52 — Set**
