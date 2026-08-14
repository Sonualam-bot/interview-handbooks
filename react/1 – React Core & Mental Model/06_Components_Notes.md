# 06_Components_Notes

## Definition

A React component is a JavaScript function that returns React Elements
describing part of the UI.

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
