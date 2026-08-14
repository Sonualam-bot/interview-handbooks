# Chapter 10 — The Scope Chain

> *"Finding a variable isn't magic. JavaScript follows a well-defined path, checking one environment after another until it either finds the identifier or gives up."*

---

# A Mystery

Consider this code.

```js
const company = "OpenAI";

function engineering() {
    const team = "Platform";

    function developer() {
        const name = "Sonu";

        console.log(name);
        console.log(team);
        console.log(company);
    }

    developer();
}

engineering();
```

How does `developer()` find three variables that live in three different places?

Does JavaScript search the entire program?

Does it keep a giant list of every variable?

Neither.

It follows a specific path every single time.

That path is called the **Scope Chain**.

---

# Becoming the JavaScript Engine

Imagine you're executing:

```js
console.log(team);
```

inside `developer()`.

Your current Environment Record contains:

```
name → "Sonu"
```

You search for `team`.

It's not there.

Do you throw an error immediately?

No.

Every Lexical Environment remembers the environment that surrounds it.

So instead of stopping, you move one step outward.

There you find:

```
team → "Platform"
```

The search ends.

No further searching is needed.

---

# What Is the Scope Chain?

The **Scope Chain** is the sequence of lexical environments JavaScript follows while resolving an identifier.

The search always begins in the current environment.

If the identifier isn't found, JavaScript moves to the next outer lexical environment.

This continues until either:

- the identifier is found, or
- the global environment is reached.

If the identifier still doesn't exist, JavaScript throws a `ReferenceError`.

---

# Think of Asking for Directions

Imagine you're looking for a book.

You first check your own desk.

If it isn't there, you ask your teammate.

If they don't have it, you ask your manager.

Then the office library.

You never skip directly to the library.

You search one level at a time.

The Scope Chain works exactly the same way.

---

# Walking Through an Example

```js
const country = "India";

function office() {
    const city = "Bangalore";

    function desk() {
        const employee = "Sonu";

        console.log(employee);
        console.log(city);
        console.log(country);
    }

    desk();
}
```

Conceptually:

```
Global Scope
│
├── country
│
└── office()
      │
      ├── city
      │
      └── desk()
            │
            └── employee
```

When resolving `country`, JavaScript searches:

1. `desk()` environment
2. `office()` environment
3. Global environment

The first successful match ends the search.

---

# Shadowing

Now consider:

```js
const language = "JavaScript";

function course() {
    const language = "TypeScript";

    console.log(language);
}

course();
```

JavaScript finds `language` in the current scope immediately.

It never continues searching outward.

The inner variable **shadows** the outer one.

The outer binding still exists.

It is simply hidden by the closer one.

---

# When the Search Fails

```js
function greet() {
    console.log(username);
}

greet();
```

JavaScript searches:

1. Current Environment Record
2. Outer Environment
3. Global Environment

Nothing is found.

Only then does it throw:

```text
ReferenceError: username is not defined
```

The engine doesn't guess.

It follows the scope chain until no environments remain.

---

# Why the Scope Chain Matters

Without an ordered search strategy:

- variable lookup would be unpredictable,
- nested functions couldn't safely access outer variables,
- closures wouldn't work,
- React callbacks couldn't access component state.

The Scope Chain gives JavaScript a deterministic way to resolve every identifier.

---

# React Connection

When a React event handler reads `count`, JavaScript doesn't magically know where `count` lives.

It walks the Scope Chain created for that render.

This is why handlers can access variables declared in the component body, and it's also why stale closures occur when a handler keeps referring to an older lexical environment.

---

# Key Takeaways

- JavaScript searches for identifiers using the Scope Chain.
- The search starts in the current lexical environment.
- If not found, the engine searches outward one environment at a time.
- The first matching binding wins.
- Inner variables can shadow outer variables.
- If no binding exists anywhere in the chain, JavaScript throws a `ReferenceError`.

---

> **Next Chapter:** *Closures — How Functions Remember Variables After Their Outer Function Has Finished*
