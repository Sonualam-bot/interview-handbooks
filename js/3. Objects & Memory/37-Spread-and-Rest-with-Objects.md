# Chapter 37 — Spread & Rest with Objects

> **Objects & Memory Handbook**

---

# What You'll Learn

- Spread Operator with Objects
- Rest Properties
- Copying Objects
- Merging Objects
- Property Overriding
- Practical Use Cases
- Common Mistakes
- Interview Questions

---

# Introduction

ES2018 introduced **object spread** and **object rest**.

Although they use the same `...` syntax, they perform opposite operations.

- **Spread** expands an object.
- **Rest** collects remaining properties.

Understanding these operators is essential before learning object copying.

---

# Spread Operator

The spread operator expands the properties of an object.

```js
const user = {
  name: "Sonu",
  age: 26
};

const copy = {
  ...user
};

console.log(copy);
```

Output:

```js
{
  name: "Sonu",
  age: 26
}
```

---

# Copying Objects

```js
const original = {
  name: "Sonu"
};

const copy = {
  ...original
};

copy.name = "Rahul";

console.log(original.name);
console.log(copy.name);
```

Output:

```text
Sonu
Rahul
```

This is a **shallow copy**.

---

# Merging Objects

```js
const personal = {
  name: "Sonu"
};

const contact = {
  city: "Bangalore",
  country: "India"
};

const profile = {
  ...personal,
  ...contact
};

console.log(profile);
```

---

# Property Overriding

If duplicate keys exist, the last value wins.

```js
const obj = {
  a: 1,
  b: 2,
  b: 5
};

console.log(obj);
```

Output:

```js
{
  a: 1,
  b: 5
}
```

The same rule applies with spread.

```js
const updated = {
  ...personal,
  name: "Rahul"
};
```

---

# Rest Properties

Rest collects the remaining properties into a new object.

```js
const user = {
  name: "Sonu",
  age: 26,
  city: "Bangalore"
};

const { name, ...rest } = user;

console.log(name);
console.log(rest);
```

Output:

```js
Sonu

{
  age: 26,
  city: "Bangalore"
}
```

---

# Practical Example

Removing a property without mutating the original object.

```js
const user = {
  id: 1,
  name: "Sonu",
  password: "secret"
};

const { password, ...publicUser } = user;

console.log(publicUser);
```

This pattern is common in backend APIs and React state updates.

---

# Spread vs Rest

| Spread | Rest |
|---------|------|
| Expands properties | Collects remaining properties |
| Used while creating objects | Used during destructuring |

Both use the same `...` syntax, but the context determines their behavior.

---

# Common Mistakes

### Assuming Spread Creates a Deep Copy

```js
const user = {
  address: {
    city: "Bangalore"
  }
};

const copy = {
  ...user
};
```

Nested objects are still shared.

We'll explore this in the next chapter.

---

### Order Matters

```js
{
  ...user,
  name: "Rahul"
}
```

`name` becomes `"Rahul"` because later properties override earlier ones.

---

# Interview Questions

### What is the difference between spread and rest?

- Spread expands an object.
- Rest collects remaining properties.

### Does spread create a deep copy?

No.

It creates a shallow copy.

### Which value wins when merging objects?

The property that appears last.

---

# Key Takeaways

- Spread copies and merges object properties.
- Rest collects remaining properties.
- Spread performs a shallow copy.
- Property order determines which values are retained.
- These operators are heavily used in React and modern JavaScript.

---

# Next Chapter

**Chapter 38 — Object Copying (Shallow vs Deep Copy)**
