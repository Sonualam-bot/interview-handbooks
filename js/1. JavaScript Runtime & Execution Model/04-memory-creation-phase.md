# Chapter 4 — The Memory Creation Phase

> *"Before a builder lays the first brick, the blueprint must already exist. JavaScript follows the same philosophy."*

---

# A Mystery

Look at this program.

```js
console.log(user);

var user = "Sonu";
```

Most beginners expect an error.

Instead, JavaScript prints:

```text
undefined
```

Now look at this.

```js
sayHello();

function sayHello() {
    console.log("Hello!");
}
```

This works even though the function appears **later** in the file.

How can JavaScript know about something it hasn't executed yet?

It seems impossible.

Unless...

JavaScript isn't reading your code the way you think it is.

---

# Becoming the JavaScript Engine

Imagine someone hands you a 5,000-line JavaScript file.

Could you execute line 1 immediately?

No.

You don't yet know:

- which variables exist,
- which functions exist,
- how much memory you'll need,
- or what identifiers you'll encounter later.

Starting execution without this knowledge would mean constantly stopping to reorganize memory.

That would be slow, inefficient, and unpredictable.

So before executing a single statement, you decide to prepare.

---

# Preparation Before Action

Think about moving into a new apartment.

Before unpacking your belongings, you first:

- unlock the door,
- inspect the rooms,
- decide where furniture will go,
- arrange the shelves.

Only then do you begin placing your belongings.

The JavaScript Engine behaves in exactly the same way.

It first prepares the workspace.

Only after preparation does it begin execution.

---

# The Two Phases of Execution

Every Execution Context goes through two distinct phases.

## Phase 1 — Memory Creation

The engine walks through the code.

It discovers declarations.

It reserves memory.

It prepares the execution environment.

But it **does not execute statements**.

---

## Phase 2 — Execution

Only after preparation is complete does JavaScript begin reading the program from top to bottom.

Assignments happen.

Expressions are evaluated.

Functions are invoked.

Values are produced.

---

# What Happens During Memory Creation?

Imagine scanning a book before reading it.

You notice:

- chapter titles,
- page numbers,
- section headings.

You haven't read the content yet.

You've simply built a mental map.

The Memory Creation Phase works similarly.

The engine scans the program and prepares everything it will need later.

---

# What Gets Prepared?

Consider:

```js
var count = 10;

let name = "Sonu";

const age = 25;

function greet() {
    return "Hello";
}
```

During Memory Creation:

- `count` receives memory and is initialized with `undefined`.
- `name` receives memory but remains uninitialized.
- `age` receives memory but remains uninitialized.
- `greet` is created as a function object.

Notice something important.

None of the assignments have happened yet.

The values `"Sonu"` and `25` have **not** been assigned.

Preparation is complete.

Execution has not yet started.

---

# Why Separate Preparation from Execution?

Imagine trying to build a house while simultaneously deciding where every room should be.

The construction would constantly stop.

Planning first makes construction faster and safer.

JavaScript follows the same strategy.

Preparation happens once.

Execution becomes simple.

---

# The First Hint of Hoisting

This preparation explains one of JavaScript's most misunderstood behaviors.

Because declarations are discovered before execution begins, it can appear as though variables and functions have been "moved to the top."

Nothing actually moves.

The engine simply prepared them earlier.

We'll study this behavior in detail in a later chapter.

---

# React Connection

Every React render creates a brand-new Execution Context.

And every Execution Context begins with a Memory Creation Phase.

That means every render first prepares its variables and function declarations before executing the component body.

This explains why each render starts with a fresh set of local bindings.

---

# Key Takeaways

- JavaScript prepares before it executes.
- Every Execution Context begins with a Memory Creation Phase.
- Declarations are discovered before statements execute.
- `var` is initialized with `undefined`.
- `let` and `const` are created but remain uninitialized.
- Function declarations are fully prepared before execution starts.

---

> **Next Chapter:** *Execution Phase — When JavaScript Finally Begins Running Your Code*
