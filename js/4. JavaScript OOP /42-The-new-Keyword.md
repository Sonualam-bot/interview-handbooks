# Chapter 42 — The `new` Keyword

> **JavaScript OOP Handbook**

---

# What You'll Learn

- Why the `new` keyword exists
- What happens internally when `new` is used
- Step-by-step execution
- Returning values from constructors
- Common mistakes
- Interview questions

---

# Introduction

In the previous chapter we learned how constructor functions define a blueprint for creating objects.

```js
function User(name) {
  this.name = name;
}
```

But what actually happens when we write:

```js
const user = new User("Sonu");
```

The `new` keyword performs several operations automatically.

---

# Syntax

```js
const object = new Constructor(arg1, arg2);
```

`Constructor` is any function intended to create objects.

---

# What Does `new` Do?

When JavaScript executes:

```js
const user = new User("Sonu");
```

it conceptually performs these steps.

### Step 1 — Create a New Empty Object

```js
{}
```

---

### Step 2 — Link the Prototype

The new object's internal `[[Prototype]]` is linked to the constructor's prototype.

```text
newObject.[[Prototype]] → User.prototype
```

This enables inheritance through the prototype chain.

---

### Step 3 — Bind `this`

Inside the constructor, `this` now refers to the newly created object.

```js
function User(name) {
  this.name = name;
}
```

Equivalent to:

```text
this = newObject
```

---

### Step 4 — Execute the Constructor

The constructor body runs.

```js
this.name = name;
```

The object is initialized.

---

### Step 5 — Return the Object

If the constructor does not explicitly return another object, JavaScript automatically returns the newly created object.

```js
const user = new User("Sonu");
```

---

# Visual Flow

```text
new User("Sonu")

↓

Create {}

↓

Link to User.prototype

↓

this → {}

↓

Execute constructor

↓

Return object
```

---

# Returning Values

## Returning a Primitive

```js
function User() {
  this.name = "Sonu";
  return 100;
}

const user = new User();

console.log(user.name);
```

Output:

```text
Sonu
```

Primitive return values are ignored.

---

## Returning an Object

```js
function User() {
  this.name = "Sonu";

  return {
    role: "Admin"
  };
}

const user = new User();

console.log(user);
```

Output:

```js
{ role: "Admin" }
```

When an object is returned explicitly, it replaces the automatically created one.

---

# Forgetting `new`

```js
function User(name) {
  this.name = name;
}

const user = User("Sonu");
```

Without `new`, the function behaves like a normal function.

In strict mode, `this` is `undefined`, causing a `TypeError`.

Always use `new` with constructor functions.

---

# Common Mistakes

### Forgetting `new`

```js
const user = User("Sonu");
```

Incorrect for constructor functions.

---

### Returning a Primitive

Primitive values are ignored when using `new`.

---

# Interview Questions

### What are the steps performed by `new`?

1. Create a new object.
2. Link its prototype.
3. Bind `this`.
4. Execute the constructor.
5. Return the object.

---

### What happens if a constructor returns an object?

That object becomes the result of the `new` expression.

---

### What happens if a constructor returns a primitive?

The primitive is ignored and the newly created object is returned.

---

# Key Takeaways

- `new` automates object creation.
- It links the object to the constructor's prototype.
- It binds `this` to the new object.
- It executes the constructor and returns the object.
- Understanding `new` makes prototypes and classes much easier to understand.

---

# Next Chapter

**Chapter 43 — Prototypes**
