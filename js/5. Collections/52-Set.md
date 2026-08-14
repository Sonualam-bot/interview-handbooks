# Chapter 52 — Set

> **Collections Handbook**

---

# What You'll Learn

- Why `Set` Was Introduced
- Creating a Set
- Adding, Checking and Removing Values
- Iterating Over a Set
- Set vs Array
- Removing Duplicates
- Time Complexity
- Common Interview Questions

---

# Introduction

Arrays can store duplicate values.

```js
const numbers = [1, 2, 2, 3, 3];
```

Sometimes duplicates are undesirable. ES6 introduced **Set**, a collection that stores **unique values**.

---

# What is a Set?

A `Set` is a collection where every value is unique.

```js
const set = new Set();

console.log(set);
```

Duplicate values are ignored.

---

# Creating a Set

```js
const numbers = new Set([1, 2, 2, 3, 3]);

console.log(numbers);
```

Output:

```text
Set(3) {1, 2, 3}
```

---

# Adding Values

Use `add()`.

```js
const fruits = new Set();

fruits.add("apple");
fruits.add("banana");
fruits.add("apple");

console.log(fruits);
```

Output:

```text
Set(2) {"apple", "banana"}
```

`add()` returns the set, allowing chaining.

---

# Checking Values

```js
console.log(fruits.has("banana"));
console.log(fruits.has("orange"));
```

Output:

```text
true
false
```

---

# Removing Values

```js
fruits.delete("banana");
```

Remove everything:

```js
fruits.clear();
```

---

# Size

```js
const set = new Set([1, 2, 3]);

console.log(set.size);
```

Output:

```text
3
```

---

# Iterating Over a Set

```js
const colors = new Set(["red", "green", "blue"]);

for (const color of colors) {
  console.log(color);
}
```

Useful methods:

```js
colors.values();
colors.keys();
colors.entries();
```

For a `Set`, `keys()` and `values()` return the same iterator.

---

# Removing Duplicates

One of the most common interview patterns.

```js
const numbers = [1, 2, 2, 3, 4, 4];

const unique = [...new Set(numbers)];

console.log(unique);
```

Output:

```js
[1, 2, 3, 4]
```

---

# Set vs Array

| Feature | Set | Array |
|---------|-----|-------|
| Duplicate Values | ❌ | ✅ |
| Indexed Access | ❌ | ✅ |
| Preserves Insertion Order | ✅ | ✅ |
| Membership Check | Fast | Linear search |

---

# Time Complexity

| Operation | Complexity |
|-----------|------------|
| `add()` | O(1) average |
| `has()` | O(1) average |
| `delete()` | O(1) average |
| Iteration | O(n) |

---

# When to Use Set

Use `Set` when:

- Values must be unique.
- Fast membership checks are needed.
- Removing duplicates from arrays.
- Tracking visited items (graphs, BFS/DFS, caches).

Use arrays when order and indexed access are more important.

---

# Common Mistakes

### Expecting Duplicate Values

```js
const set = new Set();

set.add(1);
set.add(1);

console.log(set.size);
```

Output:

```text
1
```

---

### Accessing by Index

```js
set[0];
```

`Set` is **not** index-based.

Convert to an array if indexed access is required.

```js
const arr = [...set];
```

---

# Interview Questions

### Why was `Set` introduced?

To efficiently store unique values.

---

### How do you remove duplicates from an array?

```js
const unique = [...new Set(array)];
```

---

### Can a `Set` contain objects?

Yes.

Each object reference is treated as a unique value.

---

# Key Takeaways

- `Set` stores unique values.
- Duplicate insertions are ignored.
- `add()`, `has()`, `delete()`, and `clear()` are the primary operations.
- `Set` is ideal for uniqueness and fast membership checks.
- A common interview pattern is using `Set` to remove duplicates from arrays.

---

# Next Chapter

**Chapter 53 — WeakMap & WeakSet**
