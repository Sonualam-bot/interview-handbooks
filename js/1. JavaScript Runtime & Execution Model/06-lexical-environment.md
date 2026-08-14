# Chapter 6 — Lexical Environment

> *"Creating a workspace isn't enough. The engine also needs a way to organize everything inside that workspace."*

---

# A Mystery

Consider this program.

```js
let language = "JavaScript";

function greet() {
    let message = "Hello";
    console.log(message, language);
}

greet();
```

When `greet()` runs:

- Where is `message` stored?
- Where is `language` stored?
- How does JavaScript know they belong to different places?
- Why doesn't every function share one giant collection of variables?

Somewhere inside the Execution Context, JavaScript must have a way to organize its data.

---

# Becoming the JavaScript Engine

Imagine you're the engine.

You've already created an Execution Context for `greet()`.

Now the developer declares:

```js
let message = "Hello";
```

Where should you keep it?

Then another variable appears.

```js
let count = 10;
```

Where should that go?

Soon there are parameters, helper functions, constants, and dozens of identifiers.

Throwing everything into one global bucket would be disastrous.

Different function calls would overwrite each other's data.

You need organization.

So every Execution Context gets its own organized storage system.

That system is called the **Lexical Environment**.

---

# Why the Name "Lexical"?

The word **lexical** comes from the source code itself.

JavaScript decides relationships between functions based on **where they are written**, not where they are called.

Because these relationships are determined from the program's text (its lexical structure), the storage system is called a **Lexical Environment**.

---

# A Mental Model

Imagine every function receives its own office.

Inside the office is a filing cabinet.

Every variable, parameter, and function declaration belonging to that function is stored in that cabinet.

When the function finishes, the office closes.

The cabinet disappears with it.

The office is the **Execution Context**.

The filing cabinet is the **Lexical Environment**.

---

# What Is a Lexical Environment?

A Lexical Environment is an internal structure created for every Execution Context.

Its responsibilities are:

- keeping track of identifiers,
- organizing local bindings,
- knowing where to search if a value is not found locally.

Today we'll focus on the idea that **every execution owns its own environment**.

The details of how identifiers are stored come in the next chapter.

---

# Every Function Gets Its Own Environment

```js
function counter() {
    let count = 0;
}
```

Calling:

```js
counter();
counter();
```

does **not** reuse the same environment.

Instead:

```
counter()

↓

Lexical Environment A

Destroyed

↓

counter()

↓

Lexical Environment B
```

Each call receives fresh storage.

This isolation is one of the reasons functions behave predictably.

---

# A Walk Through an Example

```js
let company = "OpenAI";

function employee() {
    let name = "Sonu";

    console.log(company);
}

employee();
```

Conceptually:

```
Global Execution Context
│
└── Lexical Environment
      └── company

↓

employee()

↓

Employee Execution Context
│
└── Lexical Environment
      └── name
```

The global variable and the local variable are intentionally stored in different environments.

This separation prevents collisions between unrelated pieces of code.

---

# Why This Matters

Imagine if every variable in every function lived in one giant shared object.

Two completely unrelated functions could accidentally overwrite each other's values.

Large applications would become impossible to maintain.

Independent lexical environments solve that problem.

Every execution receives its own private storage.

---

# A Hint About the Future

Soon you'll discover something interesting.

Each Lexical Environment also knows about the environment outside it.

That single idea eventually explains:

- scope,
- scope chains,
- closures,
- stale closures in React,
- `useCallback`,
- `useMemo`.

We're not there yet.

For now, remember only one thing:

> Every Execution Context owns its own Lexical Environment.

---

# React Connection

Every time a React component renders, JavaScript creates:

- a new Execution Context,
- a new Lexical Environment.

That is why each render gets fresh variables, fresh helper functions, and fresh event handlers.

Understanding this is the first step toward understanding closures in React.

---

# Key Takeaways

- Every Execution Context contains a Lexical Environment.
- A Lexical Environment organizes the identifiers belonging to that execution.
- Each function invocation gets a brand-new Lexical Environment.
- Different function calls never share the same lexical storage.
- The next chapter explores how identifiers are actually stored inside this environment.

---

> **Next Chapter:** *Environment Record — The Structure That Stores Variables, Parameters, and Functions*
