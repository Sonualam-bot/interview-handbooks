# Chapter 35 — Object Methods

> **Objects & Memory Handbook**

---

# What You'll Learn

- What Object Methods are
- Methods vs Properties
- The `this` Keyword
- Method Shorthand
- Borrowing Methods
- Common Mistakes
- Interview Questions

---

# Introduction

Objects don't just store data.

They can also store **behavior**.

When a property stores a function, that function is called an **object method**.

```js
const user = {
  name: "Sonu",

  greet() {
    console.log("Hello!");
  }
};

user.greet();
```

---

# Methods are Properties

A method is simply a property whose value is a function.

```js
const user = {
  greet: function () {
    console.log("Hello!");
  }
};
```

ES6 introduced the shorter syntax:

```js
const user = {
  greet() {
    console.log("Hello!");
  }
};
```

Both are equivalent.

---

# Why Use Methods?

Methods allow an object to operate on its own data.

```js
const calculator = {
  add(a, b) {
    return a + b;
  }
};

console.log(calculator.add(2, 3));
```

Objects become self-contained units containing both data and behavior.

---

# The `this` Keyword

Inside a method, `this` refers to the object before the dot.

```js
const user = {
  name: "Sonu",

  greet() {
    console.log(`Hello ${this.name}`);
  }
};

user.greet();
```

Output:

```text
Hello Sonu
```

---

# Why Not Use the Object Name?

Instead of:

```js
const user = {
  name: "Sonu",

  greet() {
    console.log(user.name);
  }
};
```

Prefer:

```js
console.log(this.name);
```

Using `this` makes the method reusable.

---

# Method Borrowing

Methods can be shared between objects.

```js
const person = {
  greet() {
    console.log(`Hello ${this.name}`);
  }
};

const user = {
  name: "Sonu"
};

user.greet = person.greet;

user.greet();
```

Output:

```text
Hello Sonu
```

The same function behaves differently because `this` changes based on how it is called.

---

# Extracting a Method

```js
const user = {
  name: "Sonu",

  greet() {
    console.log(this.name);
  }
};

const fn = user.greet;

fn();
```

In strict mode:

```text
undefined
```

The function is no longer called as a method, so `this` is lost.

We'll solve this later using `call`, `apply`, and `bind`.

---

# Arrow Functions as Methods

Avoid arrow functions for object methods.

```js
const user = {
  name: "Sonu",

  greet: () => {
    console.log(this.name);
  }
};

user.greet();
```

Arrow functions do not create their own `this`.

They inherit it from the surrounding scope.

---

# Common Mistakes

### Forgetting `this`

```js
const user = {
  name: "Sonu",

  greet() {
    console.log(name);
  }
};
```

`name` is not a local variable.

Use:

```js
console.log(this.name);
```

---

### Using Arrow Functions as Methods

Arrow functions usually behave differently than expected because they don't bind `this`.

---

# Interview Questions

### What is an object method?

A property whose value is a function.

### What does `this` refer to?

The object that invoked the method.

### Are methods copied when assigned to another object?

The function is reused; only the object used as `this` changes.

---

# Key Takeaways

- Methods are function-valued properties.
- ES6 introduced method shorthand syntax.
- `this` refers to the calling object.
- Methods can be shared across objects.
- Avoid arrow functions for object methods when `this` is required.

---

# Next Chapter

**Chapter 36 — Object Destructuring**
