# Chapter 49 — Array Methods (Part 2)

> **Collections Handbook**

---

# What You'll Learn

- `reduce()`
- `some()`
- `every()`
- `sort()`
- `reverse()`
- Choosing the Right Method
- Time Complexity
- Common Interview Questions

---

# Introduction

In the previous chapter, we learned methods used for iteration and transformation.

This chapter covers methods used for:

- Aggregation
- Validation
- Sorting
- Reversing collections

These methods appear frequently in frontend interviews and day-to-day JavaScript development.

---

# reduce()

`reduce()` combines all elements of an array into a single value.

## Syntax

```js
array.reduce(callback, initialValue)
```

The callback receives:

```js
(accumulator, currentValue, index, array)
```

---

## Example: Sum of Numbers

```js
const numbers = [1, 2, 3, 4];

const total = numbers.reduce(
  (sum, num) => sum + num,
  0
);

console.log(total);
```

Output:

```text
10
```

---

## Example: Count Occurrences

```js
const fruits = [
  "apple",
  "banana",
  "apple"
];

const counts = fruits.reduce((acc, fruit) => {
  acc[fruit] = (acc[fruit] || 0) + 1;
  return acc;
}, {});

console.log(counts);
```

Output:

```js
{
  apple: 2,
  banana: 1
}
```

---

# some()

`some()` checks whether **at least one** element satisfies a condition.

```js
const numbers = [2, 4, 6, 7];

const hasOdd = numbers.some(
  num => num % 2 !== 0
);

console.log(hasOdd);
```

Output:

```text
true
```

It stops as soon as a match is found.

---

# every()

`every()` checks whether **all** elements satisfy a condition.

```js
const numbers = [2, 4, 6];

const allEven = numbers.every(
  num => num % 2 === 0
);

console.log(allEven);
```

Output:

```text
true
```

Execution stops as soon as one element fails.

---

# sort()

`sort()` sorts the array **in place**.

```js
const numbers = [5, 2, 8, 1];

numbers.sort();

console.log(numbers);
```

Output:

```js
[1, 2, 5, 8]
```

---

## Sorting Numbers Correctly

By default, `sort()` compares values as strings.

```js
const numbers = [1, 10, 2];

numbers.sort();

console.log(numbers);
```

Output:

```js
[1, 10, 2]
```

Correct approach:

```js
numbers.sort((a, b) => a - b);
```

Ascending:

```js
[1, 2, 10]
```

Descending:

```js
numbers.sort((a, b) => b - a);
```

---

# reverse()

`reverse()` reverses the array **in place**.

```js
const arr = [1, 2, 3];

arr.reverse();

console.log(arr);
```

Output:

```js
[3, 2, 1]
```

---

# Choosing the Right Method

| Goal | Method |
|------|--------|
| Aggregate values | `reduce()` |
| Check at least one match | `some()` |
| Check every element | `every()` |
| Sort elements | `sort()` |
| Reverse order | `reverse()` |

---

# Time Complexity

| Method | Complexity |
|---------|------------|
| `reduce()` | O(n) |
| `some()` | O(n) worst case |
| `every()` | O(n) worst case |
| `sort()` | O(n log n) |
| `reverse()` | O(n) |

---

# Common Mistakes

## Forgetting an Initial Value in `reduce()`

```js
numbers.reduce((a, b) => a + b);
```

Works for non-empty arrays but can throw errors on empty arrays.

Prefer:

```js
numbers.reduce((a, b) => a + b, 0);
```

---

## Incorrect Numeric Sorting

```js
[1, 10, 2].sort();
```

This performs **lexicographic** sorting.

Use a comparison function for numeric sorting.

---

## Forgetting `sort()` Mutates the Array

```js
numbers.sort();
```

The original array is modified.

---

# Interview Questions

### What is `reduce()` used for?

To combine an array into a single value such as a sum, object, or grouped result.

---

### What is the difference between `some()` and `every()`?

- `some()` returns `true` if at least one element matches.
- `every()` returns `true` only if all elements match.

---

### Why does `sort()` require a comparison function for numbers?

Because the default implementation compares values as strings.

---

### Does `reverse()` return a new array?

No.

It mutates the existing array.

---

# Key Takeaways

- `reduce()` aggregates values.
- `some()` checks if any element matches.
- `every()` checks if all elements match.
- `sort()` mutates the array and should use a comparison function for numbers.
- `reverse()` also mutates the array.
- These methods are commonly used in React applications and coding interviews.

---

# Next Chapter

**Chapter 50 — Modern Array Methods**
