# Chapter 55 — Generators

> **Collections Handbook**

---

# What You'll Learn

- Why Generators Exist
- Generator Functions
- The `yield` Keyword
- The Generator Object
- `yield*`
- Lazy Evaluation
- Practical Use Cases
- Interview Questions

---

# Introduction

Normally, when a function is called, it runs from start to finish and returns a single value.

```js
function greet() {
  return "Hello";
}
```

Generators work differently.

They can **pause**, **resume**, and produce multiple values over time.

This makes them useful for lazy evaluation and implementing custom iterators.

---

# What is a Generator?

A generator is a special function declared using `function*`.

```js
function* numbers() {
  yield 1;
  yield 2;
  yield 3;
}
```

Calling a generator **does not execute its body immediately**.

Instead, it returns a generator object.

```js
const gen = numbers();

console.log(gen);
```

---

# The `yield` Keyword

`yield` pauses execution and returns a value.

```js
function* numbers() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = numbers();

console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
console.log(gen.next());
```

Output:

```js
{ value: 1, done: false }
{ value: 2, done: false }
{ value: 3, done: false }
{ value: undefined, done: true }
```

Each call to `next()` resumes execution from the previous `yield`.

---

# Generator Execution

Conceptually:

```text
Start

↓

yield 1

↓

Paused

↓

next()

↓

yield 2

↓

Paused

↓

next()

↓

Finished
```

Unlike normal functions, generators preserve their execution state.

---

# Generators are Iterables

Generator objects implement the iterable protocol.

```js
function* colors() {
  yield "red";
  yield "green";
  yield "blue";
}

for (const color of colors()) {
  console.log(color);
}
```

Output:

```text
red
green
blue
```

They also work with the spread operator.

```js
console.log([...colors()]);
```

---

# Returning a Value

Generators can return a final value.

```js
function* demo() {
  yield 1;
  return 2;
}

const gen = demo();

console.log(gen.next());
console.log(gen.next());
```

Output:

```js
{ value: 1, done: false }
{ value: 2, done: true }
```

The returned value is **not** included in `for...of` iteration.

---

# Delegating with `yield*`

`yield*` delegates iteration to another iterable.

```js
function* first() {
  yield 1;
  yield 2;
}

function* second() {
  yield* first();
  yield 3;
}

console.log([...second()]);
```

Output:

```js
[1, 2, 3]
```

---

# Lazy Evaluation

Generators produce values **only when requested**.

```js
function* infinite() {
  let n = 1;

  while (true) {
    yield n++;
  }
}

const gen = infinite();

console.log(gen.next().value);
console.log(gen.next().value);
console.log(gen.next().value);
```

Only the requested values are generated.

---

# Practical Use Cases

Generators are useful for:

- Custom iterators
- Lazy data generation
- Infinite sequences
- Streaming large datasets
- State machines

---

# Common Mistakes

### Expecting Immediate Execution

```js
function* demo() {
  console.log("Running");
}

demo();
```

Nothing is printed because the generator has not started.

Call `next()` to begin execution.

---

### Forgetting `yield`

Without `yield`, a generator behaves much like a normal function.

---

# Interview Questions

### What is a generator?

A special function that can pause and resume execution while producing multiple values.

---

### What does `yield` do?

It pauses execution, returns a value, and resumes when `next()` is called again.

---

### Are generators iterable?

Yes. Generator objects implement the iterable protocol.

---

### What is `yield*`?

It delegates iteration to another iterable or generator.

---

# Key Takeaways

- Generator functions are declared using `function*`.
- `yield` pauses execution and returns values one at a time.
- Calling a generator returns a generator object.
- Generator objects are both iterators and iterables.
- Generators enable lazy evaluation and efficient iteration.

---

# Next Chapter

**Chapter 56 — ES Modules**
