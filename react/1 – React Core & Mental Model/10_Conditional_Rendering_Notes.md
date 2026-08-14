# 10_Conditional_Rendering_Notes

## Definition

Conditional rendering is the process of rendering different UI based on
application state. JavaScript evaluates the condition first; React
renders the result.

------------------------------------------------------------------------

## Mental Model

``` text
State
↓
Component Executes
↓
JavaScript Evaluates Condition
↓
React Element Tree
↓
Reconciliation
↓
Commit
↓
Real DOM
```

------------------------------------------------------------------------

## Core Concepts

### JavaScript Evaluates First

``` jsx
loading ? <Spinner /> : <Dashboard />
```

-   `loading = true` → `<Spinner />`
-   `loading = false` → `<Dashboard />`

React only receives the evaluated JSX.

### Conditional Rendering Creates Different Trees

``` text
loading = true
        ↓
Spinner Tree

loading = false
        ↓
Dashboard Tree
```

Reconciliation compares these trees.

### && Operator

``` jsx
isAdmin && <AdminPanel />
```

-   `true` → renders `<AdminPanel />`
-   `false` → React receives `false` and renders nothing.

### Returning null

``` jsx
if (!showBanner) return null;
```

`null` tells React to render nothing.

------------------------------------------------------------------------

## Execution Flow

``` text
setState()
↓
React Stores State
↓
Schedules Render
↓
Component Executes Again
↓
JavaScript Evaluates Condition
↓
New React Element Tree
↓
Reconciliation
↓
Commit
↓
Real DOM
```

------------------------------------------------------------------------

## Interview Nuggets

-   JavaScript evaluates conditions.
-   React renders the evaluated result.
-   `false`, `true`, `null`, and `undefined` are ignored during
    rendering.
-   State changes create new React Element trees.

------------------------------------------------------------------------

## Common Mistakes

❌ React evaluates conditions.

✅ JavaScript evaluates conditions before React creates React Elements.

❌ `false && <Component />` creates an empty component.

✅ It evaluates to `false`, which React ignores.

------------------------------------------------------------------------

## Flashcards

**Q:** Who evaluates JSX conditions?

**A:** JavaScript.

**Q:** Why does `false && <Component />` render nothing?

**A:** Because JavaScript returns `false`, which React ignores.

**Q:** What happens after a condition changes?

**A:** React creates a new React Element tree, reconciles it with the
previous tree, then commits DOM updates.

------------------------------------------------------------------------

## 30-Second Revision

-   Conditional rendering is powered by JavaScript.
-   React receives evaluated JSX.
-   Different conditions create different React Element trees.
-   Reconciliation compares trees.
-   Commit updates the DOM.
