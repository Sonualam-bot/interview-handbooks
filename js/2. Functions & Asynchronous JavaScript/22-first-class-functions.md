# Chapter 22 — First-Class Functions

> *"Most programming languages treat functions as special. JavaScript treats them as values."*

---

# A Mystery

Consider this code.

```js
function greet() {
    console.log("Hello!");
}

const sayHello = greet;

sayHello();
```

Why does this work?

We never copied the function.

We never recreated it.

We simply assigned it to another variable.

Now consider this.

```js
function execute(fn) {
    fn();
}

execute(greet);
```

A function is being passed into another function.

Later we'll even return functions from functions.

How can a language treat executable code like ordinary data?

---

# Becoming the JavaScript Engine

Imagine you're executing:

```js
function greet() {}
```

You create a function object.

Now the developer writes:

```js
const sayHello = greet;
```

Do you copy the function?

No.

Just as assigning an object copies a reference—not the object itself—you simply make `sayHello` point to the same function object.

Functions are values.

Variables merely hold references to them.

---

# What Does "First-Class" Mean?

A language supports **first-class functions** when functions can be treated like any other value.

That means a function can be:

- stored in variables,
- stored in arrays and objects,
- passed as arguments,
- returned from other functions.

In JavaScript, functions enjoy the same flexibility as numbers, strings, and objects.

---

# Think of a Toolbox

Imagine a toolbox.

Each tool can be:

- stored on a shelf,
- handed to a coworker,
- packed into another toolbox,
- returned after use.

Functions behave similarly.

They aren't tied to one place.

They can travel through your program wherever they're needed.

---

# Storing Functions

```js
function add(a, b) {
    return a + b;
}

const operation = add;
```

Both identifiers now refer to the same function object.

Calling either one executes the same code.

```js
operation(2, 3);
add(2, 3);
```

---

# Passing Functions

Functions can become arguments.

```js
function greet() {
    console.log("Hello");
}

function run(task) {
    task();
}

run(greet);
```

Notice something subtle.

We pass:

```js
greet
```

not:

```js
greet()
```

The first passes the function itself.

The second executes it immediately.

---

# Returning Functions

Functions can also produce other functions.

```js
function createMultiplier(x) {
    return function (y) {
        return x * y;
    };
}

const double = createMultiplier(2);

console.log(double(5));
```

This works because functions are ordinary values.

Returning a function is no different from returning a number or an object.

This idea is also where closures become incredibly powerful.

---

# Higher-Order Functions

A **Higher-Order Function** is simply a function that:

- accepts another function,
- returns another function,
- or both.

Examples you already use:

```js
map()
filter()
reduce()
setTimeout()
```

Their power comes directly from first-class functions.

---

# Why This Matters

Imagine if functions couldn't be passed around.

You couldn't build:

- callbacks,
- promises,
- event listeners,
- middleware,
- hooks,
- functional programming utilities.

Modern JavaScript would look completely different.

---

# React Connection

React is built on first-class functions.

Examples include:

```jsx
<button onClick={handleClick} />
```

```jsx
items.map(item => <Card key={item.id} />)
```

```jsx
useEffect(() => {
    // side effect
}, []);
```

Callbacks, render functions, hooks, memoization, and component composition all rely on JavaScript treating functions as ordinary values.

---

# Key Takeaways

- JavaScript treats functions as first-class values.
- Functions can be stored, passed, and returned.
- Assigning a function copies its reference, not its code.
- Higher-order functions exist because functions are first-class.
- Many modern JavaScript and React patterns depend on this capability.

---

> **Next Chapter:** *Higher-Order Functions — Functions That Work With Other Functions*
