# Chapter 3 — The Call Stack

> *"Creating a workspace solves only half the problem. The engine must also remember which workspace is currently active."*

---

# A Mystery

Imagine this program:

```js
function wakeUp() {
    brushTeeth();
}

function brushTeeth() {
    makeCoffee();
}

function makeCoffee() {
    console.log("Coffee is ready.");
}

wakeUp();
```

At first, this looks straightforward.

But pause for a moment.

When `makeCoffee()` finishes, how does JavaScript know it should return to `brushTeeth()` instead of jumping back to `wakeUp()`?

And when `brushTeeth()` finishes, how does it remember to continue inside `wakeUp()`?

JavaScript cannot rely on luck.

It needs a system.

---

# Becoming the JavaScript Engine

You're the JavaScript Engine.

The program starts executing.

The global code is running.

Then you encounter:

```js
wakeUp();
```

A new Execution Context is created.

A few moments later, inside that function, you see:

```js
brushTeeth();
```

Now you have a problem.

`wakeUp()` has not finished yet.

You cannot throw away its workspace because execution must return there later.

So what do you do?

You temporarily pause it.

Then you start executing `brushTeeth()`.

Later, `brushTeeth()` calls `makeCoffee()`.

Now two functions are waiting.

How do you remember the order?

---

# The Problem JavaScript Needed to Solve

Functions can call other functions.

Those functions can call even more functions.

JavaScript must always know:

- Which function is currently executing.
- Which function should resume next.
- Which workspace belongs to which function.

This is not a variable problem.

This is an ordering problem.

---

# The Stack Idea

Imagine a stack of books on a table.

You always place a new book on the top.

You always remove the top book first.

You cannot remove the second book until the one above it is gone.

This behavior is called **Last In, First Out (LIFO).**

JavaScript realized this behavior perfectly matches nested function execution.

So it uses a stack.

---

# The Call Stack

The **Call Stack** is an internal stack that stores active Execution Contexts.

Every time a function starts:

- a new Execution Context is created,
- and pushed onto the top of the stack.

When the function finishes:

- its Execution Context is removed,
- and execution resumes with the context now at the top.

---

# Walking Through an Example

```js
function one() {
    two();
}

function two() {
    three();
}

function three() {
    console.log("Hello");
}

one();
```

Initially:

```
Global
```

After `one()`:

```
one()
Global
```

After `two()`:

```
two()
one()
Global
```

After `three()`:

```
three()
two()
one()
Global
```

Now `three()` finishes.

It leaves the stack.

```
two()
one()
Global
```

Then `two()` finishes.

```
one()
Global
```

Finally `one()` finishes.

```
Global
```

Notice something beautiful.

JavaScript never had to "remember" where to return.

The stack already remembered.

---

# Why Not a Queue?

Suppose JavaScript used a queue.

The first function added would be the first removed.

That would mean `wakeUp()` finishes before `makeCoffee()`.

Clearly impossible.

Nested function execution naturally follows LIFO.

The stack is not an arbitrary choice.

It is the only structure that naturally models function execution.

---

# When Things Go Wrong

```js
function recurse() {
    recurse();
}

recurse();
```

Each call creates another Execution Context.

Each context is pushed onto the stack.

Nothing is ever removed.

Eventually there is no room left.

JavaScript throws:

```
RangeError: Maximum call stack size exceeded
```

This is called a **Stack Overflow**.

---

# React Connection

Every React component execution creates a new Execution Context.

When components render other components, the JavaScript Engine manages them using the same Call Stack.

React does not replace the Call Stack.

It builds on top of it.

---

# Key Takeaways

- The Call Stack stores active Execution Contexts.
- It follows the Last In, First Out (LIFO) principle.
- Every function call pushes a new Execution Context.
- Every completed function removes its Execution Context.
- Stack Overflow occurs when contexts keep growing without being removed.

---

> **Next Chapter:** *Memory Creation Phase — Why JavaScript Knows About Variables Before Executing Them*
