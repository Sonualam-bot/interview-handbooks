# Chapter 21 — Arrow Functions

> *"Arrow functions look like a shorter way to write functions. But behind the new syntax lies a completely different way of thinking about function behavior."*

---

# A Mystery

Look at these two functions.

```js
function greet(name) {
    return "Hello " + name;
}
```

and

```js
const greet = (name) => {
    return "Hello " + name;
};
```

They produce exactly the same output.

So are they identical?

Now consider this example.

```js
const user = {
    name: "Sonu",

    regular() {
        console.log(this.name);
    },

    arrow: () => {
        console.log(this.name);
    }
};

user.regular();
user.arrow();
```

One prints:

```text
Sonu
```

The other prints:

```text
undefined
```

The syntax changed only slightly.

The behavior changed dramatically.

Why?

---

# Becoming the JavaScript Engine

Imagine you're creating a function object.

If the developer writes:

```js
function greet() {}
```

you create a normal function.

This function receives its own:

- `this`
- `arguments`
- `super` (when applicable)
- `new.target`

Now imagine the developer writes:

```js
const greet = () => {};
```

This time you create a different kind of function.

It intentionally **does not** create its own `this`.

Instead, it remembers the surrounding one.

That single design decision changes how arrow functions behave.

---

# Why Arrow Functions Were Introduced

Before ES6, JavaScript developers constantly wrote code like this:

```js
const self = this;

setTimeout(function () {
    console.log(self.name);
}, 1000);
```

or

```js
setTimeout(function () {
    console.log(this.name);
}.bind(this), 1000);
```

The language forced developers to work around changing `this` values.

Arrow functions solved this problem by capturing the surrounding `this` automatically.

---

# A Simpler Syntax

Arrow functions were also designed to reduce boilerplate.

Instead of:

```js
function square(x) {
    return x * x;
}
```

you can write:

```js
const square = x => x * x;
```

For simple expressions, the return value is implicit.

---

# Lexical `this`

This is the defining feature of arrow functions.

Consider:

```js
function Person() {
    this.name = "Sonu";

    setTimeout(() => {
        console.log(this.name);
    }, 1000);
}
```

The callback doesn't receive a new `this`.

Instead, it captures the `this` from `Person()`.

Conceptually:

```text
Person()

↓

this → newly created object

↓

Arrow Function

↓

inherits the same this
```

This behavior is called **lexical `this`**.

---

# No Own `arguments`

Normal functions create their own `arguments` object.

```js
function show() {
    console.log(arguments);
}
```

Arrow functions do not.

```js
const show = () => {
    console.log(arguments);
};
```

Instead, they inherit `arguments` from the nearest surrounding non-arrow function.

---

# Cannot Be Constructors

Normal functions may be called with:

```js
new User();
```

Arrow functions cannot.

```js
const User = () => {};

new User();
```

Results in:

```text
TypeError
```

Arrow functions are not constructors because they were never designed to create objects.

---

# When Should You Use Arrow Functions?

Arrow functions are excellent for:

- callbacks,
- array methods,
- promises,
- asynchronous code,
- event handlers that should inherit the surrounding `this`.

Regular functions remain the better choice for:

- object methods that depend on dynamic `this`,
- constructors,
- prototype methods.

Choosing between them is about behavior—not brevity.

---

# React Connection

Arrow functions appear everywhere in React.

Examples include:

```jsx
items.map(item => ...)
```

```jsx
onClick={() => handleClick()}
```

```jsx
const handleSubmit = () => {}
```

React developers rely heavily on lexical `this`, lexical scope, and closures.

Understanding arrow functions explains why callbacks naturally access component variables without additional binding.

---

# Key Takeaways

- Arrow functions are a distinct kind of function, not merely shorter syntax.
- They capture `this` lexically from the surrounding environment.
- They do not create their own `arguments`.
- They cannot be used as constructors with `new`.
- They are ideal for callbacks and modern React code.

---

> **Next Chapter:** *First-Class Functions — Why Functions Can Be Stored, Passed, and Returned Like Any Other Value*
