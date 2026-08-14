# Chapter 41 — Constructor Functions

> **JavaScript OOP Handbook**

---

# What You'll Learn

- Why Constructor Functions Exist
- Creating Objects with Constructor Functions
- The `this` Keyword
- Creating Multiple Objects
- Constructor Naming Convention
- Constructors vs Object Literals
- Common Mistakes
- Interview Questions

---

# Introduction

Suppose we want to create multiple users.

```js
const user1 = {
  name: "Sonu",
  age: 26
};

const user2 = {
  name: "Rahul",
  age: 24
};
```

This quickly becomes repetitive.

Constructor functions solve this problem by acting as **blueprints** for creating similar objects.

> Before ES6 classes, constructor functions were the primary way to create reusable object types.

---

# What is a Constructor Function?

A constructor function is a regular JavaScript function that is intended to create objects.

By convention, constructor names begin with a capital letter.

```js
function User(name, age) {
  this.name = name;
  this.age = age;
}
```

---

# Creating Objects

Constructor functions are called using the `new` keyword.

```js
function User(name, age) {
  this.name = name;
  this.age = age;
}

const user = new User("Sonu", 26);

console.log(user);
```

Output:

```js
User {
  name: "Sonu",
  age: 26
}
```

---

# Understanding `this`

Inside a constructor function, `this` refers to the object being created.

```js
function Car(brand) {
  this.brand = brand;
}

const car = new Car("Toyota");

console.log(car.brand);
```

Output:

```text
Toyota
```

---

# Creating Multiple Objects

```js
function User(name, age) {
  this.name = name;
  this.age = age;
}

const user1 = new User("Sonu", 26);
const user2 = new User("Rahul", 24);

console.log(user1);
console.log(user2);
```

Each call creates a new object.

---

# Adding Methods

Methods can also be added inside the constructor.

```js
function User(name) {
  this.name = name;

  this.greet = function () {
    console.log(`Hello ${this.name}`);
  };
}

const user = new User("Sonu");

user.greet();
```

This works, but every object gets its own copy of `greet()`.

A better approach is to place shared methods on the prototype, which we'll learn in later chapters.

---

# Constructor Functions vs Object Literals

Object Literal:

```js
const user = {
  name: "Sonu"
};
```

Constructor Function:

```js
function User(name) {
  this.name = name;
}

const user = new User("Sonu");
```

Use object literals for single objects and constructor functions when creating many similar objects.

---

# Common Mistakes

### Forgetting the Capitalized Name

```js
function user() {}
```

It works, but constructors are conventionally capitalized to indicate they should be called with `new`.

---

### Forgetting `new`

```js
const user = User("Sonu");
```

Without `new`, the function behaves like a normal function.

We'll explore exactly what happens in the next chapter.

---

# Interview Questions

### What is a constructor function?

A regular function used as a blueprint for creating multiple objects.

---

### Why are constructor names capitalized?

It is a JavaScript convention indicating the function should be called with `new`.

---

### Can constructor functions contain methods?

Yes, but placing methods inside the constructor creates a new copy for every object.

---

# Key Takeaways

- Constructor functions create multiple similar objects.
- They are regular functions intended to be used with `new`.
- `this` refers to the object being created.
- Constructors were the standard object-creation pattern before ES6 classes.
- Shared methods are better placed on the prototype.

---

# Next Chapter

**Chapter 42 — The `new` Keyword**
