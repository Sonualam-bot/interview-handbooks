# Chapter 54 — Iterables & Iterators

> **Collections Handbook**

---

# What You'll Learn

- What Iterables are
- What Iterators are
- Iterable Protocol
- Iterator Protocol
- `Symbol.iterator`
- Built-in Iterables
- Creating a Custom Iterable
- Interview Questions

---

# Introduction

Many JavaScript features work with collections:

- `for...of`
- Spread operator (`...`)
- Destructuring
- `Array.from()`

These features rely on the **Iterable Protocol**.

---

# What is an Iterable?

An **iterable** is any object that implements the **Iterable Protocol** by exposing a `Symbol.iterator` method.

```js
const arr = [1, 2, 3];

console.log(typeof arr[Symbol.iterator]);
```

Output:

```text
function
```

This is why arrays work with `for...of`.

---

# Built-in Iterables

The following are iterable:

- Arrays
- Strings
- Maps
- Sets
- Typed Arrays

Examples:

```js
for (const ch of "JS") {
  console.log(ch);
}
```

---

# What is an Iterator?

An **iterator** is an object returned by `Symbol.iterator()`.

It exposes a `next()` method.

```js
const arr = [10, 20];

const iterator = arr[Symbol.iterator]();

console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
```

Output:

```js
{ value: 10, done: false }
{ value: 20, done: false }
{ value: undefined, done: true }
```

---

# Iterator Protocol

An iterator must implement:

```js
next()
```

Each call returns:

```js
{
  value,
  done
}
```

- `value` → current item
- `done` → whether iteration has finished

---

# Iterable Protocol

An iterable must implement:

```js
[Symbol.iterator]()
```

which returns an iterator.

Conceptually:

```text
Iterable
   │
   ▼
Symbol.iterator()
   │
   ▼
Iterator
   │
   ▼
next()
```

---

# Creating a Custom Iterable

```js
const range = {
  start: 1,
  end: 3,

  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;

    return {
      next() {
        if (current <= end) {
          return {
            value: current++,
            done: false
          };
        }

        return {
          value: undefined,
          done: true
        };
      }
    };
  }
};

for (const n of range) {
  console.log(n);
}
```

Output:

```text
1
2
3
```

---

# Where Iterables are Used

The iterable protocol powers:

```js
for (const x of arr) {}
```

```js
const copy = [...arr];
```

```js
const chars = Array.from("Hello");
```

```js
const [a, b] = arr;
```

---

# Common Mistakes

### Confusing Iterables with Iterators

- Iterable → has `Symbol.iterator()`
- Iterator → has `next()`

---

### Using `for...of` on Non-Iterables

```js
const obj = { a: 1 };

for (const x of obj) {}
```

Throws:

```text
TypeError: obj is not iterable
```

---

# Interview Questions

### What is the difference between an iterable and an iterator?

An iterable produces an iterator.

An iterator produces values through `next()`.

---

### What does `Symbol.iterator` do?

It returns an iterator object used by JavaScript iteration features.

---

### Why does `for...of` work on arrays?

Because arrays implement the iterable protocol.

---

# Key Takeaways

- Iterables implement `Symbol.iterator()`.
- Iterators implement `next()`.
- `for...of`, spread syntax, destructuring, and `Array.from()` rely on the iterable protocol.
- Arrays, strings, maps, and sets are built-in iterables.
- You can create your own iterable objects using `Symbol.iterator`.

---

# Next Chapter

**Chapter 55 — Generators**
