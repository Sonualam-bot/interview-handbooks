# Chapter 50 — Modern Array Methods

> **Collections Handbook**

---

# What You'll Learn

- Why Modern Array Methods Were Introduced
- `includes()`
- `at()`
- `flat()`
- `flatMap()`
- `fill()`
- `copyWithin()`
- Time Complexity
- Common Interview Questions

---

# Introduction

Over the years, JavaScript has introduced several new array methods to solve common problems more elegantly.

These methods reduce boilerplate code and improve readability.

Many of them are frequently used in modern React and Node.js applications.

---

# includes()

`includes()` checks whether an array contains a particular value.

## Syntax

```js
array.includes(value)
```

Example:

```js
const fruits = ["apple", "banana", "mango"];

console.log(fruits.includes("banana"));
console.log(fruits.includes("orange"));
```

Output:

```text
true
false
```

Unlike `indexOf()`, `includes()` returns a boolean and correctly handles `NaN`.

```js
const numbers = [1, 2, NaN];

console.log(numbers.includes(NaN));
```

Output:

```text
true
```

---

# at()

`at()` accesses elements using positive or negative indexes.

Example:

```js
const arr = [10, 20, 30];

console.log(arr.at(0));
console.log(arr.at(-1));
console.log(arr.at(-2));
```

Output:

```text
10
30
20
```

Before `at()`:

```js
arr[arr.length - 1];
```

With `at()`:

```js
arr.at(-1);
```

Much cleaner.

---

# flat()

`flat()` flattens nested arrays.

```js
const arr = [1, 2, [3, 4]];

console.log(arr.flat());
```

Output:

```js
[1, 2, 3, 4]
```

Flatten multiple levels:

```js
const arr = [1, [2, [3, [4]]]];

console.log(arr.flat(2));
```

Output:

```js
[1, 2, 3, [4]]
```

Flatten completely:

```js
arr.flat(Infinity);
```

---

# flatMap()

`flatMap()` combines `map()` and `flat(1)`.

```js
const words = ["hello world", "javascript"];

const result = words.flatMap(
  word => word.split(" ")
);

console.log(result);
```

Output:

```js
["hello", "world", "javascript"]
```

Equivalent to:

```js
words.map(...).flat();
```

but more efficient and readable.

---

# fill()

`fill()` replaces elements with a static value.

```js
const arr = [1, 2, 3, 4];

arr.fill(0);

console.log(arr);
```

Output:

```js
[0, 0, 0, 0]
```

Fill part of an array:

```js
const arr = [1, 2, 3, 4];

arr.fill(9, 1, 3);

console.log(arr);
```

Output:

```js
[1, 9, 9, 4]
```

---

# copyWithin()

`copyWithin()` copies a portion of an array to another location **within the same array**.

It **mutates** the array.

```js
const arr = [1, 2, 3, 4, 5];

arr.copyWithin(0, 3);

console.log(arr);
```

Output:

```js
[4, 5, 3, 4, 5]
```

Syntax:

```js
array.copyWithin(target, start, end)
```

No new array is created.

---

# Which Methods Mutate?

| Method | Mutates? |
|---------|----------|
| `includes()` | ❌ |
| `at()` | ❌ |
| `flat()` | ❌ |
| `flatMap()` | ❌ |
| `fill()` | ✅ |
| `copyWithin()` | ✅ |

Knowing which methods mutate the original array is a common interview topic.

---

# Time Complexity

| Method | Complexity |
|---------|------------|
| `includes()` | O(n) |
| `at()` | O(1) |
| `flat()` | O(n) |
| `flatMap()` | O(n) |
| `fill()` | O(n) |
| `copyWithin()` | O(n) |

---

# Common Mistakes

## Using `indexOf()` Instead of `includes()`

Instead of:

```js
if (arr.indexOf(value) !== -1) {
}
```

Prefer:

```js
if (arr.includes(value)) {
}
```

---

## Assuming `flat()` Changes the Original Array

```js
const result = arr.flat();
```

`flat()` returns a **new array**.

The original remains unchanged.

---

## Forgetting `fill()` Mutates

```js
arr.fill(0);
```

The original array is modified.

---

# Interview Questions

### What is the difference between `map()` and `flatMap()`?

`flatMap()` performs a `map()` followed by a one-level flatten.

---

### Which methods mutate the array?

- `fill()`
- `copyWithin()`

---

### Why use `at(-1)`?

It provides a clean way to access elements from the end of an array.

---

### What is the advantage of `includes()` over `indexOf()`?

It returns a boolean directly and correctly handles `NaN`.

---

# Key Takeaways

- `includes()` checks for element existence.
- `at()` supports negative indexing.
- `flat()` flattens nested arrays.
- `flatMap()` combines mapping and flattening.
- `fill()` and `copyWithin()` mutate the original array.
- These methods simplify many common array operations.

---

# Next Chapter

**Chapter 51 — Map**
