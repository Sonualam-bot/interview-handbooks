# Chapter 48 — Array Methods (Part 1)

> **Collections Handbook**

---

# What You'll Learn

- Why Array Methods Exist
- `forEach()`
- `map()`
- `filter()`
- `find()`
- `findIndex()`
- When to Use Each Method
- Time Complexity
- Common Interview Questions

---

# Introduction

JavaScript arrays provide many built-in methods that make working with collections easier.

Instead of writing manual `for` loops, modern JavaScript encourages using expressive array methods.

Understanding **which method to use and why** is a common interview topic.

---

# forEach()

`forEach()` executes a callback once for every element.

```js
const numbers = [1, 2, 3];

numbers.forEach((num) => {
  console.log(num);
});
```

Output:

```text
1
2
3
```

### Characteristics

- Iterates over every element.
- Returns `undefined`.
- Cannot be chained.
- Best for side effects (logging, DOM updates, etc.).

---

# map()

`map()` transforms every element and returns a **new array**.

```js
const numbers = [1, 2, 3];

const doubled = numbers.map((num) => num * 2);

console.log(doubled);
```

Output:

```js
[2, 4, 6]
```

The original array is not modified.

### Use Cases

- Data transformation
- Preparing UI data
- React rendering

---

# filter()

`filter()` returns a new array containing only elements that satisfy a condition.

```js
const numbers = [1, 2, 3, 4, 5];

const even = numbers.filter((num) => num % 2 === 0);

console.log(even);
```

Output:

```js
[2, 4]
```

---

# find()

`find()` returns the **first matching element**.

```js
const users = [
  { id: 1, name: "Sonu" },
  { id: 2, name: "Rahul" }
];

const user = users.find((u) => u.id === 2);

console.log(user);
```

Output:

```js
{ id: 2, name: "Rahul" }
```

If no element matches, it returns:

```js
undefined
```

---

# findIndex()

`findIndex()` returns the index of the first matching element.

```js
const numbers = [10, 20, 30];

const index = numbers.findIndex((n) => n === 20);

console.log(index);
```

Output:

```text
1
```

If no match exists:

```text
-1
```

---

# Comparison

| Method | Returns |
|---------|---------|
| `forEach()` | `undefined` |
| `map()` | New array |
| `filter()` | New array |
| `find()` | First matching element |
| `findIndex()` | Index of first matching element |

---

# Time Complexity

| Method | Complexity |
|---------|------------|
| `forEach()` | O(n) |
| `map()` | O(n) |
| `filter()` | O(n) |
| `find()` | O(n) |
| `findIndex()` | O(n) |

All methods may inspect each element once.

---

# Common Mistakes

### Using `map()` for Side Effects

```js
numbers.map((n) => console.log(n));
```

Use `forEach()` instead.

---

### Expecting `filter()` to Return One Element

```js
const result = numbers.filter(...);
```

`filter()` always returns an array.

If you need one element, use `find()`.

---

### Forgetting `find()` Returns `undefined`

Always handle the case where no match exists.

---

# Interview Questions

### What is the difference between `map()` and `forEach()`?

- `map()` returns a new array.
- `forEach()` returns `undefined`.

---

### When should you use `filter()` instead of `find()`?

Use `filter()` when multiple matches are possible.

Use `find()` when only the first match is needed.

---

### Does `map()` modify the original array?

No.

It creates and returns a new array.

---

# Key Takeaways

- `forEach()` performs side effects.
- `map()` transforms data.
- `filter()` selects matching elements.
- `find()` returns the first matching value.
- `findIndex()` returns the position of the first match.
- These methods are heavily used in React and JavaScript interviews.

---

# Next Chapter

**Chapter 49 — Array Methods (Part 2)**
