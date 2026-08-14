# Chapter 36 — Object Destructuring

> **Objects & Memory Handbook**

---

# What You'll Learn

- What Object Destructuring is
- Why Destructuring Exists
- Basic Destructuring
- Renaming Variables
- Default Values
- Nested Destructuring
- Function Parameter Destructuring
- Common Mistakes
- Interview Questions

---

# Introduction

Objects often contain many properties.

Without destructuring:

```js
const user = {
  name: "Sonu",
  age: 26,
  city: "Bangalore"
};

const name = user.name;
const age = user.age;
```

JavaScript provides **destructuring** to extract properties more concisely.

---

# Basic Destructuring

```js
const user = {
  name: "Sonu",
  age: 26
};

const { name, age } = user;

console.log(name);
console.log(age);
```

Output:

```text
Sonu
26
```

The variable names must match the property names.

---

# Renaming Variables

You can rename extracted values.

```js
const user = {
  name: "Sonu"
};

const { name: fullName } = user;

console.log(fullName);
```

Output:

```text
Sonu
```

---

# Default Values

Defaults are used when a property is `undefined`.

```js
const user = {
  name: "Sonu"
};

const { name, city = "Bangalore" } = user;

console.log(city);
```

Output:

```text
Bangalore
```

---

# Nested Destructuring

Objects can contain other objects.

```js
const user = {
  name: "Sonu",
  address: {
    city: "Bangalore",
    country: "India"
  }
};

const {
  address: { city }
} = user;

console.log(city);
```

Output:

```text
Bangalore
```

---

# Destructuring in Function Parameters

A common interview pattern.

```js
function greet({ name }) {
  console.log(`Hello ${name}`);
}

greet({
  name: "Sonu"
});
```

This is frequently used in React props and API handlers.

---

# Combining Defaults and Destructuring

```js
function createUser({
  name = "Anonymous",
  age = 18
} = {}) {
  return { name, age };
}
```

Passing an empty object prevents runtime errors.

---

# Common Mistakes

### Property Doesn't Exist

```js
const user = {
  name: "Sonu"
};

const { age } = user;

console.log(age);
```

Output:

```text
undefined
```

---

### Renaming Incorrectly

```js
const { name } = user;
```

creates `name`.

```js
const { name: fullName } = user;
```

creates `fullName`.

---

### Deep Destructuring Without Checks

```js
const {
  address: { city }
} = {};
```

Throws because `address` is `undefined`.

Use defaults or optional access patterns when appropriate.

---

# Interview Questions

### What is object destructuring?

A syntax for extracting object properties into variables.

---

### Can destructuring rename variables?

Yes.

```js
const { name: fullName } = user;
```

---

### When are default values used?

Only when the property value is `undefined`.

---

# Key Takeaways

- Destructuring extracts object properties.
- Variable names normally match property names.
- Properties can be renamed.
- Default values make destructuring safer.
- Function parameter destructuring is widely used in modern JavaScript and React.

---

# Next Chapter

**Chapter 37 — Spread & Rest with Objects**
