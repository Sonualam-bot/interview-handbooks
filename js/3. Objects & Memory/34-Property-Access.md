# Chapter 34 — Property Access

> **Objects & Memory Handbook**

---

# What You'll Learn

- Accessing Object Properties
- Dot Notation
- Bracket Notation
- Dynamic Property Access
- Property Existence
- Optional Chaining
- Common Mistakes
- Interview Questions

---

# Introduction

After creating an object, the next step is accessing and modifying its properties.

JavaScript provides two primary ways to access properties:

- Dot notation (`.`)
- Bracket notation (`[]`)

Understanding when to use each is a common interview topic.

---

# Dot Notation

Dot notation is the simplest and most commonly used way to access properties.

```js
const user = {
  name: "Sonu",
  age: 26
};

console.log(user.name);
console.log(user.age);
```

Output:

```text
Sonu
26
```

Dot notation is concise and easy to read.

---

# Updating Properties

Properties can be updated using dot notation.

```js
const user = {
  name: "Sonu"
};

user.name = "Rahul";

console.log(user.name);
```

Output:

```text
Rahul
```

---

# Adding New Properties

Objects are dynamic.

```js
const user = {};

user.city = "Bangalore";

console.log(user);
```

Output:

```js
{
  city: "Bangalore"
}
```

---

# Bracket Notation

Bracket notation accesses properties using a string.

```js
const user = {
  name: "Sonu"
};

console.log(user["name"]);
```

Output:

```text
Sonu
```

---

# Why Bracket Notation?

Bracket notation is useful when:

- Property names contain spaces.
- Property names are stored in variables.
- Property names are computed dynamically.

---

# Dynamic Property Access

```js
const key = "age";

const user = {
  age: 26
};

console.log(user[key]);
```

Output:

```text
26
```

Using dot notation:

```js
console.log(user.key);
```

would look for a property literally named `"key"`.

---

# Properties with Special Characters

```js
const user = {
  "first name": "Sonu"
};

console.log(user["first name"]);
```

Dot notation cannot be used here.

---

# Checking Property Existence

Use the `in` operator.

```js
const user = {
  name: "Sonu"
};

console.log("name" in user);
console.log("age" in user);
```

Output:

```text
true
false
```

---

# Optional Chaining

Accessing deeply nested properties can cause errors.

```js
const user = {};

console.log(user.address.city);
```

Output:

```text
TypeError
```

Optional chaining prevents this.

```js
console.log(user.address?.city);
```

Output:

```text
undefined
```

Execution stops safely if an intermediate property is `null` or `undefined`.

---

# Deleting Properties

Use the `delete` operator.

```js
const user = {
  name: "Sonu",
  age: 26
};

delete user.age;

console.log(user);
```

Output:

```js
{
  name: "Sonu"
}
```

---

# Common Mistakes

### Using Dot Notation with Variables

```js
const key = "name";

user.key;
```

This searches for `"key"`.

Correct:

```js
user[key];
```

---

### Accessing Missing Nested Properties

```js
user.profile.name;
```

May throw an error.

Prefer:

```js
user.profile?.name;
```

---

# Interview Questions

### What is the difference between dot and bracket notation?

- Dot notation uses the property name directly.
- Bracket notation evaluates an expression to determine the property name.

---

### When should bracket notation be used?

- Dynamic property names
- Property names with spaces
- Computed property access

---

### What does optional chaining do?

It safely accesses nested properties without throwing an error if an intermediate value is `null` or `undefined`.

---

# Key Takeaways

- Dot notation is the most common way to access properties.
- Bracket notation supports dynamic property names.
- The `in` operator checks whether a property exists.
- Optional chaining prevents errors when accessing nested properties.
- Objects remain dynamic throughout their lifetime.

---

# Next Chapter

**Chapter 35 — Object Methods**
