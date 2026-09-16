# 06_Components_Notes

## Definition

A React component is a JavaScript function that returns React Elements
describing part of the UI.

------------------------------------------------------------------------

## Deep Dive `[NEW]`

### Why Functions, and Why "Pure"

A component being *just a function of props/state* is what makes an
instance's UI fully re-derivable — call it again with the same
inputs, get the same output, every time. This directly extends `UI =
f(state)` (see What is React) down to the individual component level:
each component is its own small `f`.

Purity is enforced by convention, not by the language, because React
reserves the right to call your component function multiple times, at
unpredictable moments, skip calling it, or call it with stale props
during concurrent rendering — none of which is safe if the function
has side effects tied to being called exactly once per "real" update.

### Definition vs Instance, Concretely

`function Button() {}` is a definition — it exists once, at import
time. `<Button />` written three times in JSX creates three
*elements*, and when React renders those elements it creates three
separate **fiber nodes** — React's internal bookkeeping structure, one
per position in the tree — each with its own slot for state, effects,
and a pointer back to the DOM node it produced. "Instance" below means
"fiber node," not an object you construct yourself.

This is also *why* isolated state works: state isn't stored on the
component function (there's only one function, shared by all three
buttons) — it's stored on each fiber, keyed by position in the tree.
That's exactly why moving a component to a different position in the
tree can reset its state even though "the same component" is still
rendering.

### Traced Example

``` jsx
function Counter() {
  const [n, setN] = useState(0);
  return <button onClick={() => setN(n + 1)}>{n}</button>;
}

<Counter />   // fiber A, own memoizedState slot for n
<Counter />   // fiber B, own memoizedState slot for n
```

Clicking the first button calls `setN` on fiber A only. React
re-executes the `Counter` function for fiber A, reads fiber A's
stored `n`, computes `n + 1`, and re-renders only that fiber's
subtree. Fiber B's stored `n` is untouched — this is the actual
mechanism behind "each instance owns its own state," not just a rule
to memorize.

------------------------------------------------------------------------

## Mental Model

``` text
Component Definition
        ↓
<Component />
        ↓
React Element
        ↓
React Executes Component
        ↓
Returns More React Elements
        ↓
Virtual DOM
        ↓
Reconciliation
        ↓
Commit
        ↓
Real DOM
```

------------------------------------------------------------------------

## Core Concepts

### Components are Functions

-   Components are ordinary JavaScript functions.
-   They return React Elements, not HTML.

### Component Definition vs Instance

One definition:

``` jsx
function Button() {}
```

Many instances:

``` jsx
<Button />
<Button />
```

Each instance has its own props, state and lifecycle.

### Component Tree

``` text
App
├── Navbar
├── Dashboard
│   ├── Sidebar
│   └── Content
└── Footer
```

### Pure Components

Given the same props and state, a component should produce the same UI
during rendering.

### Initial Mount vs Re-render

Initial Mount: - Execute component - Build React Element tree - Commit
to DOM

Re-render: - State/props change - Execute component again - Build new
React Element tree - Reconciliation - Commit

------------------------------------------------------------------------

## Interview Nuggets

-   Components are JavaScript functions.
-   Components return React Elements.
-   React executes components recursively.
-   One component definition can create many instances.
-   Each instance has isolated local state.

------------------------------------------------------------------------

## Common Mistakes

❌ Components return HTML.

✅ Components return React Elements.

❌ Component instances share state.

✅ Each instance owns its own state.

❌ Reconciliation happens on first mount.

✅ Reconciliation compares previous and new trees during updates.

------------------------------------------------------------------------

## Flashcards

**Q:** What does a component return?

**A:** React Elements.

**Q:** Why doesn't one Counter update another?

**A:** Each Counter is a separate component instance with isolated
state.

------------------------------------------------------------------------

## 30-Second Revision

-   Components are functions.
-   Components return React Elements.
-   React executes components recursively.
-   One definition → many instances.
-   Each instance has isolated state.
-   Reconciliation happens on updates.
