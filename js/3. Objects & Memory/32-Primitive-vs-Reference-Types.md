# Chapter 32 — Primitive vs Reference Types

> **Objects & Memory Handbook**

---

# What You'll Learn

- Primitive values
- Reference values
- How JavaScript stores them
- Pass by Value
- Pass by Sharing
- Why objects behave differently
- Common interview questions

---

# Introduction

One of the most common JavaScript interview questions is:

> **Why does changing one object affect another, while changing a number doesn't?**

The answer lies in the difference between **Primitive** and **Reference** types.

This chapter builds directly on the JavaScript Memory Model.

---

# JavaScript Data Types

JavaScript data types are divided into two categories.

## Primitive Types

- String
- Number
- Boolean
- Undefined
- Null
- Symbol
- BigInt

Example:

```js
let age = 25;
let name = "Sonu";
```

---

## Reference Types

- Object
- Array
- Function
- Date
- Map
- Set
- RegExp

Example:

```js
const user = {
  name: "Sonu"
};
```

---

# Primitive Values

Primitive values represent the actual data.

```js
let a = 10;
let b = a;

b = 20;

console.log(a);
console.log(b);
```

Output:

```text
10
20
```

Changing `b` does not affect `a` because the value is copied.

---

# Reference Values

Reference values store a reference to an object.

```js
const user1 = {
  name: "Sonu"
};

const user2 = user1;

user2.name = "Rahul";

console.log(user1.name);
```

Output:

```text
Rahul
```

Both variables refer to the same object.

---

# Conceptual Memory

```text
Stack

user1 ─────┐
           │
user2 ─────┘

           │
           ▼

Heap

{
  name: "Rahul"
}
```

Only one object exists.

---

# Pass by Value

JavaScript always passes function arguments by value.

For primitive values, the copied value is the actual data.

```js
function increment(x) {
  x++;
}

let num = 5;

increment(num);

console.log(num);
```

Output:

```text
5
```

---

# Pass by Sharing

For objects, the copied value is the reference.

```js
function rename(user) {
  user.name = "Rahul";
}

const person = {
  name: "Sonu"
};

rename(person);

console.log(person.name);
```

Output:

```text
Rahul
```

The function receives a copy of the reference, not a new object.

---

# Reassigning Doesn't Affect the Original

```js
function reset(user) {
  user = {
    name: "New User"
  };
}

const person = {
  name: "Sonu"
};

reset(person);

console.log(person.name);
```

Output:

```text
Sonu
```

The local parameter now references a different object, while the caller still points to the original one.

---

# Common Mistakes

❌ Thinking objects are passed by reference.

JavaScript is **pass-by-value**.

For objects, the copied value happens to be a reference.

---

# Interview Questions

### Are objects passed by reference?

No.

JavaScript is pass-by-value.

For objects, the copied value is the reference.

---

### Why does changing one object affect another?

Because both variables reference the same object in memory.

---

### Why don't primitive values behave the same way?

Because primitives are copied directly.

---

# Key Takeaways

- Primitive values are copied directly.
- Objects are accessed through references.
- Variables never share primitive values.
- Multiple variables can refer to the same object.
- Understanding this distinction is essential before learning objects and object copying.

---

# Next Chapter

**Chapter 33 — Objects**
