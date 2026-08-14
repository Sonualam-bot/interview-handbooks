# Chapter 12 — Garbage Collection

> *"Creating memory is only half the job. A language also needs to know when that memory is no longer needed."*

---

# A Mystery

Look at this code.

```js
function createUser() {
    let name = "Sonu";
    let role = "Frontend Engineer";

    console.log(name);
}

createUser();
```

When `createUser()` finishes, what happens to `name` and `role`?

Do they stay in memory forever?

If every variable from every function remained in memory, every JavaScript program would eventually consume all available RAM.

Somehow, the engine must know when memory can safely be reclaimed.

---

# Becoming the JavaScript Engine

Imagine you're the engine.

A function finishes executing.

Its Execution Context has been removed from the Call Stack.

Now you inspect its Lexical Environment.

You ask one simple question:

> "Can anything still reach these variables?"

If the answer is **no**, the memory can be released.

If the answer is **yes**, the memory must remain alive.

This decision is the heart of garbage collection.

---

# What Is Garbage Collection?

**Garbage Collection (GC)** is the automatic process by which JavaScript reclaims memory that is no longer reachable by the running program.

Unlike languages where developers manually free memory, JavaScript performs this work automatically.

The goal is simple:

- keep useful memory,
- reclaim useless memory.

---

# Think of a Library

Imagine a library.

Books currently being read stay on the desks.

Books nobody is using are returned to the shelves.

Eventually, damaged or obsolete books may be removed entirely.

Memory works in a similar way.

Objects that are still being used remain in memory.

Objects that nothing can reach anymore become candidates for cleanup.

---

# Reachability

Modern JavaScript engines don't ask:

> "Was this variable created?"

They ask:

> "Can this value still be reached?"

Consider:

```js
let user = {
    name: "Sonu"
};
```

As long as `user` refers to that object, it is reachable.

Now:

```js
user = null;
```

If no other reference exists, the object becomes unreachable.

The garbage collector is now free to reclaim its memory.

---

# Closures and Reachability

Earlier, we learned that closures preserve lexical environments.

Consider:

```js
function counter() {
    let count = 0;

    return function () {
        count++;
    };
}

const increment = counter();
```

Even though `counter()` has finished, `count` is still reachable through the returned function.

The garbage collector leaves it alone.

Only after `increment` itself becomes unreachable can the preserved environment be collected.

---

# What Gets Collected?

Conceptually, the garbage collector can reclaim:

- objects,
- arrays,
- functions,
- preserved lexical environments,
- other values that are no longer reachable.

Exactly *when* this happens is intentionally unspecified.

JavaScript guarantees correctness, not the precise timing.

---

# You Don't Control the Timing

Developers sometimes ask:

> "When does garbage collection run?"

The answer is:

You don't know.

Different JavaScript engines use sophisticated algorithms and heuristics.

The collector runs whenever the engine decides it is appropriate.

Your code should never depend on a garbage collection cycle happening at a specific moment.

---

# Why Automatic Memory Management Matters

Imagine manually freeing memory after every object.

Forgetting even once could cause a memory leak.

Freeing memory too early could crash the program.

Automatic garbage collection removes an entire category of programming errors while allowing developers to focus on application logic.

---

# React Connection

When a React component unmounts, its local variables don't disappear immediately.

If nothing still references them—such as timers, event listeners, or closures—they become unreachable and are eventually reclaimed by the garbage collector.

Understanding reachability helps explain why forgotten subscriptions or lingering references can cause memory leaks in React applications.

---

# Key Takeaways

- JavaScript automatically manages memory through Garbage Collection.
- Memory is reclaimed when values become unreachable.
- Closures can keep variables alive by maintaining references.
- Developers cannot control exactly when garbage collection runs.
- Reachability—not age—determines whether memory can be freed.

---

> **Next Chapter:** *Hoisting — Why JavaScript Appears to Know About Declarations Before It Executes Them*
