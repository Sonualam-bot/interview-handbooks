# Chapter 57 — Closures Revisited

> **Performance Patterns Handbook**

---

# What You'll Learn

- Why Revisit Closures?
- Closure Recap
- Lexical Environment
- Closures as a Design Pattern
- Data Privacy
- Function Factories
- Real-world Use Cases
- Performance Considerations
- Common Interview Questions

---

# Introduction

Closures are one of the most important concepts in JavaScript.

You have already learned what a closure is. In this handbook, we'll focus on **how closures are used in real-world applications and interviews**.

Most JavaScript interview questions involving functions, callbacks, event handlers, memoization, currying, and debouncing rely on closures.

---

# Quick Recap

A closure is created when a function remembers variables from its outer lexical scope even after the outer function has finished executing.

```js
function outer() {
  const message = "Hello";

  return function inner() {
    console.log(message);
  };
}

const greet = outer();

greet();
```

Output:

```text
Hello
```

The inner function remembers `message` through a closure.

---

# Lexical Environment

A closure captures the **lexical environment** in which it was created.

```js
function counter() {
  let count = 0;

  return function () {
    count++;
    return count;
  };
}

const increment = counter();

console.log(increment());
console.log(increment());
console.log(increment());
```

Output:

```text
1
2
3
```

Even after `counter()` has finished executing, the variable `count` remains available.

---

# Closures for Data Privacy

Closures allow variables to remain private.

```js
function createBankAccount() {
  let balance = 1000;

  return {
    deposit(amount) {
      balance += amount;
    },

    getBalance() {
      return balance;
    }
  };
}

const account = createBankAccount();

account.deposit(500);

console.log(account.getBalance());
```

Output:

```text
1500
```

The `balance` variable cannot be accessed directly.

---

# Function Factories

Closures make it easy to create specialized functions.

```js
function multiply(multiplier) {
  return function (value) {
    return value * multiplier;
  };
}

const double = multiply(2);
const triple = multiply(3);

console.log(double(5));
console.log(triple(5));
```

Output:

```text
10
15
```

Each returned function closes over a different `multiplier`.

---

# Closures in Event Handlers

Closures are commonly used in browser applications.

```js
function createLogger(message) {
  return function () {
    console.log(message);
  };
}

button.addEventListener(
  "click",
  createLogger("Button clicked")
);
```

The event handler remembers the original message.

---

# Closures and Asynchronous Code

Closures are heavily used with timers.

```js
function delayedGreeting(name) {
  setTimeout(() => {
    console.log(`Hello ${name}`);
  }, 1000);
}

delayedGreeting("Sonu");
```

The callback remembers `name` even after `delayedGreeting()` has returned.

---

# Performance Considerations

Closures keep referenced variables alive.

If a closure holds large objects unnecessarily, those objects remain reachable and cannot be garbage collected.

Use closures intentionally and avoid capturing more data than necessary.

---

# Common Mistakes

## Creating Closures Inside Loops

Using `var` may lead to unexpected results.

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

Output:

```text
3
3
3
```

Using `let` creates a new binding for each iteration.

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

Output:

```text
0
1
2
```

---

# Interview Questions

### What is a closure?

A closure is a function that remembers variables from its lexical scope even after the outer function has finished executing.

---

### Why are closures useful?

They enable:

- Data privacy
- Function factories
- Event handlers
- Asynchronous callbacks
- Memoization

---

### Can closures cause memory leaks?

Yes.

If closures keep unnecessary references to large objects, those objects cannot be garbage collected.

---

# Key Takeaways

- Closures preserve lexical scope.
- They enable private state and reusable function factories.
- Closures are widely used in asynchronous JavaScript.
- Understanding closures is essential for advanced JavaScript interview questions.

---

# Next Chapter

**Chapter 58 — Currying**
