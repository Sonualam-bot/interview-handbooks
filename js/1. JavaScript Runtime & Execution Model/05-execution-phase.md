# Chapter 5 — The Execution Phase

> *"Preparation is finished. The stage is set. Now the JavaScript Engine finally begins doing the work you asked it to do."*

---

# A Mystery

Consider this program.

```js
let firstName = "Sonu";
let lastName = "Alam";

function fullName() {
    return firstName + " " + lastName;
}

console.log(fullName());
```

You already know that before execution begins, JavaScript prepares memory.

But preparation alone cannot produce output.

Nothing has been printed.

Nothing has been calculated.

Nothing has been assigned.

So when does all of that actually happen?

---

# Becoming the JavaScript Engine

Imagine you've just completed the Memory Creation Phase.

Every declaration has been discovered.

Memory has been reserved.

Functions are ready.

Variables exist.

Now you stand at the beginning of the file.

For the first time, you begin reading the program exactly as the developer wrote it.

One statement.

One instruction.

One line at a time.

This is the Execution Phase.

---

# From Preparation to Action

Think of a theater.

Before the audience arrives:

- the stage is built,
- the lights are installed,
- the microphones are tested,
- the actors get into costume.

None of that is the performance.

Only when the curtain rises does the story actually begin.

The Memory Creation Phase prepares the stage.

The Execution Phase performs the play.

---

# The Execution Phase

During the Execution Phase, JavaScript executes the program from top to bottom.

This is when the engine:

- assigns values,
- evaluates expressions,
- calls functions,
- creates new Execution Contexts,
- returns values,
- produces output.

Everything visible to the programmer happens here.

---

# Walking Through an Example

```js
let a = 10;
let b = 20;

let sum = a + b;

console.log(sum);
```

Let's become the engine.

### Line 1

```js
let a = 10;
```

The binding for `a` already exists.

Now JavaScript initializes it with the value `10`.

---

### Line 2

```js
let b = 20;
```

The binding for `b` already exists.

JavaScript stores `20` inside it.

---

### Line 3

```js
let sum = a + b;
```

This line requires work.

The engine:

1. reads `a`,
2. finds `10`,
3. reads `b`,
4. finds `20`,
5. performs the addition,
6. stores `30` inside `sum`.

Only now does `sum` actually contain a value.

---

### Line 4

```js
console.log(sum);
```

The engine reads the value stored in `sum`.

It sends `30` to the console.

Only now does the programmer finally see output.

---

# Function Calls During Execution

Now consider:

```js
function greet(name) {
    return "Hello " + name;
}

const message = greet("Sonu");
```

Everything is prepared.

Then execution reaches:

```js
greet("Sonu");
```

The engine cannot continue inside the current Execution Context.

Instead it:

1. pauses the current execution,
2. creates a new Execution Context,
3. executes `greet`,
4. receives the returned value,
5. resumes the previous execution.

This is where the Call Stack becomes important.

---

# Preparation vs Execution

Imagine reading a recipe.

Reading the ingredient list is preparation.

Actually chopping vegetables and cooking is execution.

Likewise:

**Memory Creation Phase**

- discovers declarations,
- prepares memory.

**Execution Phase**

- performs assignments,
- computes values,
- executes functions.

One prepares.

The other performs.

---

# Why Execute Line by Line?

Suppose JavaScript tried to execute every statement simultaneously.

Variables might be used before they receive values.

Functions could return before parameters exist.

Dependencies would become impossible to manage.

Sequential execution keeps the language predictable.

One instruction finishes before the next begins.

---

# React Connection

Every React render goes through the same journey.

First:

- Memory Creation Phase.

Then:

- Execution Phase.

During execution the component:

- evaluates hooks,
- creates JSX,
- computes derived values,
- creates event handlers,
- returns the UI.

React never changes JavaScript's execution model.

It relies on it.

---

# Key Takeaways

- The Execution Phase begins after memory preparation.
- JavaScript executes statements from top to bottom.
- Assignments happen during execution, not during preparation.
- Function calls create new Execution Contexts while execution is in progress.
- Every React render also follows this two-phase execution model.

---

> **Next Chapter:** *Lexical Environment — Where JavaScript Stores Variables While Your Code Runs*
