# Chapter 20 — `this`

> *"Most variables are resolved by where they are written. `this` follows a different set of rules entirely."*

---

# A Mystery

Look at these two examples.

```js
const user = {
    name: "Sonu",
    greet() {
        console.log(this.name);
    }
};

user.greet();
```

Output:

```text
Sonu
```

Now compare it with:

```js
const greet = user.greet;

greet();
```

The exact same function runs.

Yet the result changes.

Why?

Nothing inside the function changed.

The only difference is **how the function was called**.

---

# Becoming the JavaScript Engine

Imagine you've reached:

```js
user.greet();
```

Before executing the function, you ask an important question.

> "Who is making this call?"

The answer is:

```text
user
```

So inside the new Execution Context, you bind:

```text
this → user
```

Now imagine:

```js
greet();
```

This time there is no owning object in the call expression.

The function body is identical.

But the call site is different.

So the value of `this` is different too.

---

# What Is `this`?

`this` is a special keyword whose value is determined **when a function is called**, not when it is defined.

Unlike ordinary variables, `this` is **not** resolved through the Scope Chain.

Instead, JavaScript determines its value using the calling syntax.

---

# Think of a Remote Control

Imagine a universal TV remote.

The remote always has the same buttons.

What changes is **which television** it is currently controlling.

The buttons don't change.

The target does.

`this` behaves similarly.

The function stays the same.

The object it operates on depends on how it is invoked.

---

# Method Calls

```js
const car = {
    brand: "Tesla",

    showBrand() {
        console.log(this.brand);
    }
};

car.showBrand();
```

Here the call is made through `car`.

Conceptually:

```text
this → car
```

So:

```js
this.brand
```

becomes:

```js
car.brand
```

---

# Plain Function Calls

```js
function greet() {
    console.log(this);
}

greet();
```

In modern JavaScript modules and strict mode:

```text
this → undefined
```

In older non-strict browser scripts, `this` becomes the global object.

This historical behavior is one reason strict mode became important.

---

# Constructor Calls

When using:

```js
function User(name) {
    this.name = name;
}

const person = new User("Sonu");
```

JavaScript creates a new object.

Then:

```text
this → newly created object
```

Every assignment to `this` builds that new instance.

---

# Arrow Functions

Arrow functions follow a completely different rule.

```js
const user = {
    name: "Sonu",

    greet: () => {
        console.log(this.name);
    }
};
```

Arrow functions do **not** create their own `this`.

Instead, they capture the surrounding `this` from the lexical environment where they were created.

This makes them especially useful for callbacks where preserving the surrounding context is desirable.

---

# Why `this` Exists

Imagine writing:

```js
car.start();
bike.start();
truck.start();
```

Without `this`, every function would need to know the object's name in advance.

Instead, JavaScript allows the same function to operate on whichever object invoked it.

This makes methods reusable.

---

# React Connection

Modern React function components rarely rely on `this`.

Hooks replaced most class-based patterns.

However, understanding `this` remains important because:

- older React class components use it extensively,
- many JavaScript libraries still rely on it,
- interview questions frequently cover its behavior,
- you'll encounter it when reading legacy code.

---

# Key Takeaways

- `this` is determined at call time.
- `this` is not resolved using the Scope Chain.
- Method calls bind `this` to the calling object.
- Plain function calls in strict mode receive `undefined`.
- Constructor calls bind `this` to the newly created object.
- Arrow functions inherit `this` lexically instead of creating their own.

---

> **Next Chapter:** *Arrow Functions — Why They Changed More Than Just Syntax*
