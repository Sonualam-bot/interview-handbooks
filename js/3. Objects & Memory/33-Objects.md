# Chapter 33 — Objects

> **Objects & Memory Handbook**

---

# What You'll Learn

- What an Object is
- Why Objects Exist
- Creating Objects
- Properties and Methods
- Dynamic Nature of Objects
- Nested Objects
- Object Identity
- Common Interview Questions

---

# Why Do We Need Objects?

Primitive values store only a single piece of data.

```js
const name = "Sonu";
const age = 26;
const city = "Bangalore";
```

As related data grows, managing separate variables becomes difficult.

Objects allow us to group related data into a single value.

---

# What is an Object?

An object is a collection of **key–value pairs**.

```js
const user = {
  name: "Sonu",
  age: 26,
  city: "Bangalore"
};
```

Here:

- `name`, `age`, and `city` are **properties**.
- `"Sonu"`, `26`, and `"Bangalore"` are their values.

---

# Object Literals

The most common way to create an object is with an object literal.

```js
const car = {
  brand: "Toyota",
  year: 2024
};
```

---

# Properties

Properties describe the state of an object.

```js
const book = {
  title: "JavaScript Handbook",
  pages: 350
};

console.log(book.title);
console.log(book.pages);
```

---

# Methods

When a property stores a function, it is called a **method**.

```js
const user = {
  name: "Sonu",

  greet() {
    console.log(`Hello ${this.name}`);
  }
};

user.greet();
```

Methods represent behavior.

---

# Objects are Dynamic

Properties can be added, updated, or removed at runtime.

```js
const user = {};

user.name = "Sonu";
user.age = 26;

delete user.age;

console.log(user);
```

Objects are mutable by default.

---

# Nested Objects

Objects can contain other objects.

```js
const user = {
  name: "Sonu",
  address: {
    city: "Bangalore",
    country: "India"
  }
};

console.log(user.address.city);
```

Nested objects help model real-world data.

---

# Object Identity

Two objects with identical contents are still different objects.

```js
const a = { x: 1 };
const b = { x: 1 };

console.log(a === b);
```

Output:

```text
false
```

Equality compares object identity, not contents.

We'll study this in detail in **Chapter 39 — Object Equality**.

---

# Real-World Example

```js
const employee = {
  id: 101,
  name: "Sonu",
  department: "Engineering",
  skills: ["React", "Node.js"]
};
```

Objects are commonly used to represent users, products, orders, API responses, and application state.

---

# Common Mistakes

### Confusing Objects with JSON

JavaScript object:

```js
const user = {
  name: "Sonu"
};
```

JSON:

```json
{
  "name": "Sonu"
}
```

JSON requires double-quoted keys and is a data interchange format.

---

# Interview Questions

### What is an object?

A collection of key–value pairs used to represent related data.

### Can objects contain functions?

Yes. Function-valued properties are called methods.

### Are objects mutable?

Yes. Properties can be added, modified, and removed unless restricted.

### Are two identical objects equal?

No.

```js
{} === {}
```

returns `false` because they are different objects.

---

# Key Takeaways

- Objects group related data.
- Objects store properties and methods.
- Objects are mutable and dynamic.
- Nested objects model complex data.
- Equality compares object identity, not structure.

---

# Next Chapter

**Chapter 34 — Property Access**
