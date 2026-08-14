# Chapter 43 — Prototypes

> **JavaScript OOP Handbook**

---

# What You'll Learn

- What a Prototype is
- Why JavaScript Uses Prototypes
- How Prototypes Enable Code Reuse
- The `prototype` Property
- Prototype vs `[[Prototype]]`
- Common Mistakes
- Interview Questions

---

# Introduction

Suppose we create multiple users using a constructor.

```js
function User(name) {
  this.name = name;

  this.greet = function () {
    console.log(`Hello ${this.name}`);
  };
}
```

Every object gets its **own copy** of `greet()`.

If we create 10,000 users, JavaScript creates 10,000 identical functions.

This wastes memory.

JavaScript solves this using **prototypes**.

---

# What is a Prototype?

A prototype is an object that stores properties and methods **shared** by all instances created from a constructor.

Instead of copying methods into every object, JavaScript stores them once on the prototype.

---

# The `prototype` Property

Every regular function automatically has a `prototype` property.

```js
function User() {}

console.log(User.prototype);
```

Output (conceptually):

```js
{
  constructor: User
}
```

This object is used only when the function is called with `new`.

---

# Sharing Methods

Instead of writing:

```js
function User(name) {
  this.name = name;

  this.greet = function () {
    console.log(`Hello ${this.name}`);
  };
}
```

Write:

```js
function User(name) {
  this.name = name;
}

User.prototype.greet = function () {
  console.log(`Hello ${this.name}`);
};
```

Now every instance shares the same method.

---

# Creating Objects

```js
const user1 = new User("Sonu");
const user2 = new User("Rahul");

user1.greet();
user2.greet();
```

Although both objects can call `greet()`, only **one copy** of the function exists.

---

# Visual Representation

```text
user1
  │
  ├── name
  │
  ▼
User.prototype
  │
  └── greet()
```

```text
user2
  │
  ├── name
  │
  ▼
User.prototype
  │
  └── greet()
```

Both objects share the same prototype object.

---

# `prototype` vs `[[Prototype]]`

These two are often confused.

### `prototype`

- Exists on constructor functions.
- Used when creating new objects.

```js
User.prototype
```

### `[[Prototype]]`

- Exists on objects.
- Points to the object's prototype.

Conceptually:

```text
User.prototype

        ▲
        │

user.[[Prototype]]
```

We'll explore `[[Prototype]]` in the next chapter.

---

# Why Prototypes Matter

Without prototypes:

- More memory usage
- Duplicate methods
- Slower object creation

With prototypes:

- Shared methods
- Better memory efficiency
- Prototype-based inheritance

---

# Common Mistakes

### Confusing `prototype` and `__proto__`

They are **not** the same thing.

`prototype` belongs to functions.

`__proto__` exposes an object's internal prototype.

---

### Adding Methods Inside the Constructor

```js
this.greet = function () {};
```

Creates one function per object.

Prefer:

```js
User.prototype.greet = function () {};
```

---

# Interview Questions

### What is a prototype?

An object that stores properties and methods shared by all instances of a constructor.

---

### Why use prototypes?

To share methods instead of creating duplicate copies for every object.

---

### Does every object have a `prototype` property?

No.

Constructor functions have a `prototype` property.

Objects have an internal `[[Prototype]]`.

---

# Key Takeaways

- Prototypes enable method sharing.
- Every constructor function has a `prototype` object.
- Instances created with `new` link to that prototype.
- Prototypes improve memory usage and support inheritance.
- Understanding prototypes is essential before learning `__proto__` and the prototype chain.

---

# Next Chapter

**Chapter 44 — `__proto__`**
