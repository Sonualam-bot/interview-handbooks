# Chapter 46 — Classes

> **JavaScript OOP Handbook**

---

# What You'll Learn

- Why ES6 Classes were introduced
- Declaring Classes
- Constructors
- Instance Methods
- Static Methods
- Getters and Setters
- Classes and Prototypes
- Common Mistakes
- Interview Questions

---

# Introduction

Before ES6, JavaScript developers primarily used **constructor functions** and
**prototypes** to create reusable object types.

```js
function User(name) {
  this.name = name;
}

User.prototype.greet = function () {
  console.log(`Hello ${this.name}`);
};
```

ES6 introduced the `class` syntax to make object-oriented code easier to read.

> **Important:** Classes do **not** introduce a new inheritance model.
> JavaScript is still prototype-based. Classes are syntactic sugar over
> constructor functions and prototypes.

---

# Declaring a Class

```js
class User {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}
```

Create an instance:

```js
const user = new User("Sonu", 26);

console.log(user);
```

Output:

```js
User { name: "Sonu", age: 26 }
```

---

# The Constructor

The `constructor()` method initializes a new object.

```js
class User {
  constructor(name) {
    this.name = name;
  }
}
```

It is automatically executed whenever `new` is used.

---

# Instance Methods

Methods declared inside a class are placed on the class prototype.

```js
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log(`Hello ${this.name}`);
  }
}

const user = new User("Sonu");
user.greet();
```

Only one copy of `greet()` exists regardless of how many instances are created.

---

# Static Methods

Static methods belong to the class itself, not its instances.

```js
class MathUtil {
  static add(a, b) {
    return a + b;
  }
}

console.log(MathUtil.add(2, 3));
```

Trying to call a static method on an instance throws an error.

---

# Getters and Setters

Classes support getters and setters.

```js
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }

  get area() {
    return this.width * this.height;
  }
}

const rect = new Rectangle(10, 20);

console.log(rect.area);
```

---

# Classes Use Prototypes

Although the syntax is different, classes still use prototypes internally.

```js
class User {
  greet() {}
}

const user = new User();

console.log(
  Object.getPrototypeOf(user) === User.prototype
);
```

Output:

```text
true
```

---

# Classes vs Constructor Functions

| Constructor Functions | Classes |
|-----------------------|---------|
| Function syntax | `class` syntax |
| Methods added manually | Methods declared inside the class |
| Prototype-based | Prototype-based |
| Requires `new` | Requires `new` |

---

# Common Mistakes

### Calling a Class Without `new`

```js
class User {}

User();
```

Output:

```text
TypeError: Class constructor User cannot be invoked without 'new'
```

---

### Assuming Classes Replace Prototypes

Classes are only a cleaner syntax.

Prototype-based inheritance still powers JavaScript behind the scenes.

---

# Interview Questions

### Are JavaScript classes a new inheritance model?

No. They are syntactic sugar over constructor functions and prototypes.

---

### Where are class methods stored?

On the class prototype.

---

### Do classes require `new`?

Yes. Calling a class without `new` throws a `TypeError`.

---

# Key Takeaways

- ES6 classes provide a cleaner syntax for creating objects.
- Classes still rely on prototypes internally.
- Constructors initialize new instances.
- Instance methods are shared through the prototype.
- Static methods belong to the class itself.

---

# Handbook Complete ✅

You have completed the **JavaScript OOP Handbook**.

Next Handbook:

**JavaScript Functions & `this` Handbook**
