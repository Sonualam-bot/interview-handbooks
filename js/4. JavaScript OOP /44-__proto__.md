# Chapter 44 — `__proto__`

> **JavaScript OOP Handbook**

---

# What You'll Learn

- What `__proto__` is
- `__proto__` vs `prototype`
- How Objects are Linked
- Property Lookup
- `Object.getPrototypeOf()`
- `Object.setPrototypeOf()`
- Common Mistakes
- Interview Questions

---

# Introduction

In the previous chapter, we learned that constructor functions have a `prototype` object.

When an object is created using `new`, JavaScript links the object to that prototype.

One way to observe this relationship is through `__proto__`.

> **Note:** `__proto__` is a legacy accessor. Modern JavaScript prefers `Object.getPrototypeOf()` and `Object.setPrototypeOf()`.

---

# What is `__proto__`?

Every ordinary object has an internal slot named `[[Prototype]]`.

`__proto__` is a legacy accessor that exposes this internal prototype.

```js
const user = {
  name: "Sonu"
};

console.log(user.__proto__);
```

Conceptually:

```text
user
 │
 ▼
[[Prototype]]

↑
__proto__
```

---

# How is `__proto__` Created?

```js
function User(name) {
  this.name = name;
}

const user = new User("Sonu");
```

Conceptually:

```text
User.prototype
      ▲
      │
user.__proto__
```

The `new` keyword links the object's internal prototype to `User.prototype`.

---

# `prototype` vs `__proto__`

These names look similar but represent different things.

| `prototype` | `__proto__` |
|-------------|-------------|
| Property on constructor functions | Property on objects |
| Used when creating new objects | Refers to an object's prototype |
| Exists before object creation | Exists after object creation |

Example:

```js
function User() {}

const user = new User();

console.log(User.prototype === user.__proto__);
```

Output:

```text
true
```

---

# Property Lookup

When JavaScript cannot find a property on an object, it checks the object's prototype.

```js
function User(name) {
  this.name = name;
}

User.prototype.greet = function () {
  console.log(`Hello ${this.name}`);
};

const user = new User("Sonu");

user.greet();
```

JavaScript finds `greet()` through `user.__proto__`.

---

# Modern APIs

Instead of:

```js
user.__proto__;
```

Use:

```js
Object.getPrototypeOf(user);
```

To change an object's prototype:

```js
Object.setPrototypeOf(user, anotherObject);
```

These APIs are clearer and part of the modern standard.

---

# Why Avoid `__proto__`?

Although widely supported, `__proto__` is a legacy feature.

For new code, prefer:

- `Object.getPrototypeOf()`
- `Object.setPrototypeOf()`

Interviewers often ask about `__proto__` because it helps explain how prototype inheritance works.

---

# Common Mistakes

### Confusing `prototype` and `__proto__`

```js
User.prototype
```

belongs to the constructor.

```js
user.__proto__
```

belongs to the instance.

---

### Modifying `__proto__` Directly

Changing an object's prototype at runtime is possible but can negatively impact performance.

Prefer designing prototype relationships during object creation.

---

# Interview Questions

### What is `__proto__`?

A legacy accessor that exposes an object's internal `[[Prototype]]`.

---

### Is `__proto__` the same as `prototype`?

No.

`prototype` belongs to constructor functions.

`__proto__` belongs to objects.

---

### What is the modern alternative to `__proto__`?

`Object.getPrototypeOf()` and `Object.setPrototypeOf()`.

---

# Key Takeaways

- Every ordinary object has an internal `[[Prototype]]`.
- `__proto__` exposes that internal prototype.
- `prototype` and `__proto__` are different concepts.
- Modern code should prefer `Object.getPrototypeOf()`.
- Understanding `__proto__` prepares you for the prototype chain.

---

# Next Chapter

**Chapter 45 — Prototype Chain**
