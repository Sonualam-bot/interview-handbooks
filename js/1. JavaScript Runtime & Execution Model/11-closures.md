# Chapter 11 — Closures

> *"Functions don't just remember their code. Sometimes they remember the world in which they were created."*

---

# A Mystery

Consider this program.

```js
function outer() {
    let count = 0;

    function increment() {
        count++;
        console.log(count);
    }

    return increment;
}

const counter = outer();

counter();
counter();
counter();
```

The output is:

```text
1
2
3
```

This seems impossible.

`outer()` has already finished.

Its Execution Context should be gone.

Its local variable `count` should have disappeared.

So how can `increment()` still access it?

---

# Becoming the JavaScript Engine

Imagine you're executing `outer()`.

You create an Execution Context.

Inside it lives:

```text
count → 0
increment → Function Object
```

Eventually you reach:

```js
return increment;
```

Normally, once a function finishes, its Execution Context is removed from the Call Stack.

But before destroying everything, you notice something important.

The returned function still depends on `count`.

If you destroy the Lexical Environment now, the returned function will never work again.

So instead of deleting that environment, you keep it alive.

---

# What Is a Closure?

A **closure** is a function together with the Lexical Environment in which it was created.

The function remembers the variables that were in scope when it was defined.

Those variables continue to exist for as long as the function can still access them.

---

# Think of a Backpack

Imagine every function leaves home carrying a backpack.

Inside the backpack are references to everything the function may need later.

When the function travels somewhere else, the backpack travels with it.

No matter where the function is eventually called, it still has access to what it packed.

A closure works in the same way.

---

# Walking Through the Example

```js
function outer() {
    let count = 0;

    return function () {
        count++;
        console.log(count);
    };
}
```

When `outer()` finishes, the returned function still references `count`.

Instead of destroying the surrounding Lexical Environment, JavaScript preserves it.

Each time the returned function runs:

```
count → 0

↓

count → 1

↓

count → 2

↓

count → 3
```

The variable survives because something still needs it.

---

# Closures Are About Reachability

JavaScript doesn't keep variables alive forever.

It keeps them alive only while they are still reachable.

If no function references the surrounding Lexical Environment anymore, it becomes eligible for garbage collection.

Closures extend the lifetime of data—but only when necessary.

---

# Multiple Closures

Every call creates a completely new closure.

```js
const counterA = outer();
const counterB = outer();

counterA();
counterA();

counterB();
```

Output:

```text
1
2
1
```

Each returned function has its own preserved Lexical Environment.

They never share the same `count`.

---

# Why Closures Exist

Without closures:

- callbacks couldn't remember local variables,
- event handlers would lose access to component state,
- timers would forget their data,
- factories couldn't create private state.

Closures make functions self-contained.

They carry the context they need wherever they go.

---

# React Connection

Closures are everywhere in React.

Every event handler:

```js
<button onClick={() => setCount(count + 1)} />
```

captures the variables from the render in which it was created.

This is also the reason stale closures happen.

The handler remembers the values from an older render—not because React forgot to update them, but because closures preserve lexical environments.

---

# Key Takeaways

- A closure is a function plus its surrounding Lexical Environment.
- Closures allow functions to access variables after the outer function has finished.
- JavaScript preserves environments only while they are still reachable.
- Every invocation creates a new closure with its own independent state.
- Closures power callbacks, event handlers, factories, hooks, and much more.

---

> **Next Chapter:** *Garbage Collection — How JavaScript Decides When Memory Can Be Freed*
