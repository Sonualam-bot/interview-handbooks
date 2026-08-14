# Chapter 47 — Arrays Revisited

> **Collections Handbook**

---

# What You'll Learn

- Are Arrays Really Arrays?
- Arrays are Objects
- Internal Representation
- Dense vs Sparse Arrays
- Array-like Objects
- Time Complexity
- Common Interview Questions

---

# Introduction

Arrays are one of the most frequently used data structures in JavaScript.

Although they look different from objects, **arrays are actually specialized objects**.

Understanding this explains many JavaScript behaviors and interview questions.

---

# Arrays are Objects

```js
const arr = [10, 20, 30];

console.log(typeof arr);
```

Output:

```text
object
```

Arrays inherit from `Array.prototype`, which in turn inherits from `Object.prototype`.

---

# How Arrays Store Data

Conceptually:

```text
Array

0 → 10
1 → 20
2 → 30
length → 3
```

Indexes are simply property names.

```js
const arr = ["a", "b"];

console.log(arr["0"]);
console.log(arr[0]);
```

Both produce the same result.

---

# The `length` Property

```js
const arr = [1, 2, 3];

console.log(arr.length);
```

Output:

```text
3
```

The `length` property is automatically updated as elements are added or removed.

---

# Dense Arrays

A dense array has values at most indexes.

```js
const arr = [1, 2, 3, 4];
```

Dense arrays are the most performant representation.

---

# Sparse Arrays

```js
const arr = [];

arr[5] = "Hello";

console.log(arr);
console.log(arr.length);
```

Output:

```text
[ <5 empty items>, "Hello" ]
6
```

Sparse arrays contain empty slots and may have different performance characteristics.

---

# Array-like Objects

Some objects look like arrays but are **not** arrays.

Example:

```js
const arrayLike = {
  0: "A",
  1: "B",
  length: 2
};
```

They have indexed properties and a `length`, but do not inherit from `Array.prototype`.

Convert them using:

```js
Array.from(arrayLike);
```

---

# Time Complexity

| Operation | Complexity |
|-----------|------------|
| Access by index | O(1) |
| Update by index | O(1) |
| push() | O(1) (amortized) |
| pop() | O(1) |
| shift() | O(n) |
| unshift() | O(n) |
| Searching | O(n) |

---

# Common Mistakes

### Arrays are not primitive values

```js
typeof []
```

returns:

```text
object
```

---

### Empty Slots vs `undefined`

```js
const arr = [];
arr[2] = undefined;
```

is different from:

```js
const arr = [];
arr[2] = "A";
delete arr[2];
```

The second creates an empty slot.

---

# Interview Questions

### Are JavaScript arrays objects?

Yes. Arrays are specialized objects with numeric property keys.

### Why is `typeof []` equal to `"object"`?

Because arrays inherit from `Object`.

### How can you check if a value is an array?

```js
Array.isArray(value);
```

---

# Key Takeaways

- Arrays are specialized objects.
- Indexes are property names.
- Arrays have a special `length` property.
- Dense arrays are preferred over sparse arrays.
- Array-like objects are not true arrays.

---

# Next Chapter

**Chapter 48 — Array Methods (Part 1)**
