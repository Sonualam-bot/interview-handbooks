# 09_Events_Notes

## Definition

Events are user interactions (clicks, typing, submissions, etc.) that
React responds to by executing event handlers.

------------------------------------------------------------------------

## Mental Model

``` text
User Interaction
      ↓
Event Handler
      ↓
State Update
      ↓
React Stores State
      ↓
Schedules Render
      ↓
Component Executes Again
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

## Core Concepts

### Event Handlers

React expects a function reference.

``` jsx
onClick={handleClick}
```

Not:

``` jsx
onClick={handleClick()}
```

------------------------------------------------------------------------

### Function Reference vs Function Call

-   `handleClick` → Pass the function.
-   `handleClick()` → Execute immediately during rendering.

------------------------------------------------------------------------

### What Happens on Click?

1.  Browser dispatches a click event.
2.  React invokes the registered handler.
3.  Handler calls `setState()`.
4.  React stores the new state.
5.  React schedules a render.
6.  Component executes again.
7.  New React Element tree is created.
8.  Reconciliation compares old vs new trees.
9.  Commit updates the DOM.

------------------------------------------------------------------------

### Synthetic Events

React wraps browser events in a consistent event API (SyntheticEvent).

------------------------------------------------------------------------

## Interview Nuggets

-   Events do not update the DOM directly.
-   Event handlers are functions.
-   State updates start React's rendering pipeline.
-   React uses camelCase event names (`onClick`, `onChange`).

------------------------------------------------------------------------

## Common Mistakes

❌ `onClick={handleClick()}`

✅ `onClick={handleClick}`

❌ Events directly manipulate the DOM.

✅ Events trigger state updates, which trigger rendering.

------------------------------------------------------------------------

## Flashcards

**Q:** Why pass a function reference to `onClick`?

**A:** So React can call it later when the event occurs.

**Q:** What happens after `setState()` in an event handler?

**A:** React stores state, schedules a render, creates a new React
Element tree, performs reconciliation, and commits DOM updates.

------------------------------------------------------------------------

## 30-Second Revision

-   Events represent user interactions.
-   Pass function references.
-   Events trigger state updates.
-   State updates trigger rendering.
-   Reconciliation computes minimal changes.
-   Commit updates the real DOM.
