# Chapter 2 — Execution Context

> *"Every great system begins by preparing before it starts working. JavaScript is no different."*

---

# A Mystery

Imagine you're writing your first calculator.

```js
function add(a, b) {
    return a + b;
}

add(10, 20);
```

At first glance, it seems simple.

The function is called.

The numbers are added.

The answer is returned.

But if you slow the process down, dozens of questions appear.

- Where are `a` and `b` stored?
- Where does `return` know to go back?
- Where does `a + b` exist while it's being calculated?
- If another function is called, how are they kept separate?

There has to be a temporary workspace created for this function.

JavaScript cannot execute code without one.

---

# Becoming the JavaScript Engine

Imagine you are no longer the programmer.

You are the JavaScript Engine.

A developer hands you this program.

```js
function greet(name) {
    const message = "Hello " + name;
    return message;
}

greet("Sonu");
```

Could you immediately execute the first statement?

No.

Before execution begins you must answer several questions.

Where will the parameter `name` live?

Where will `message` be stored?

How will you remember that after `greet()` finishes you should continue executing the code that called it?

If another function starts while this one is still running, how will you keep them separate?

Without a plan, execution would quickly become chaos.

So before executing a function, you create a dedicated workspace.

That workspace is called an **Execution Context**.

---

# The Problem JavaScript Needed to Solve

A JavaScript program is rarely a single line.

Functions call other functions.

Those functions create variables.

Those variables disappear when the function finishes.

JavaScript needs a way to isolate every function call so that one execution never accidentally interferes with another.

The solution is simple:

> Every execution gets its own workspace.

---

# The Execution Context

An **Execution Context** is the internal workspace JavaScript creates whenever global code starts or a function is invoked.

Think of it as the function's private office.

Everything needed while that function is running lives inside that office.

When the function finishes, the office is closed and removed.

---

# A Mental Model

Imagine a customer service company.

Every new customer receives a support ticket.

That ticket contains:

- the customer's details,
- the current status,
- notes,
- and everything required to complete the request.

Two customers never share the same ticket.

Likewise, two function calls never share the same Execution Context.

Even if they call the same function.

---

# First Function Call

```js
function square(x) {
    return x * x;
}

square(5);
```

When `square(5)` is invoked, JavaScript creates a brand-new Execution Context.

Inside it:

- `x` receives the argument `5`.
- Space exists for local variables.
- JavaScript tracks the current line being executed.
- JavaScript remembers where to return after the function completes.

Only after this preparation does execution begin.

When the function returns, the entire Execution Context is destroyed.

---

# Two Calls, Two Worlds

```js
square(2);
square(8);
```

Although both calls execute the same code, JavaScript creates two completely independent execution contexts.

The second call does not reuse the first one's variables.

This isolation is one of the reasons JavaScript behaves predictably.

---

# What's Inside an Execution Context?

We'll explore each part in later chapters, but conceptually it contains:

- A place to store variables.
- Information about the current execution.
- A lexical environment.
- Information required to resume execution after the function returns.

Think of today's chapter as discovering that the workspace exists.

The next chapters will unpack everything inside it.

---

# React Connection

A React component is just a JavaScript function.

Every render of a component creates a brand-new Execution Context.

This single fact explains why:

- every render has fresh local variables,
- every render creates new functions,
- every render creates new closures.

Almost every React hook eventually traces back to this idea.

---

# Key Takeaways

- JavaScript prepares a workspace before executing code.
- That workspace is called an Execution Context.
- Every function invocation receives its own Execution Context.
- Execution Contexts isolate function executions from one another.
- React components follow the exact same execution model because they are ordinary JavaScript functions.

---

> **Next Chapter:** *The Call Stack — How JavaScript Keeps Track of Multiple Execution Contexts*
