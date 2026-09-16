# 09_Events_Notes

## Definition

Events are user interactions (clicks, typing, submissions, etc.) that
React responds to by executing event handlers.

------------------------------------------------------------------------

## Deep Dive `[NEW]`

### Why SyntheticEvent Exists At All

Historically, browsers implemented DOM events inconsistently (event
object shape, propagation quirks). React wraps native events in a
`SyntheticEvent` with a consistent, cross-browser API, so
`event.target.value` behaves the same regardless of which browser
fired it. This has architectural consequences: older React versions
*pooled* (reused) event objects for performance, which is why
accessing `event` asynchronously used to require `event.persist()`.
Modern React no longer pools, but "React wraps events" was never just
style consistency — it's what let that optimization exist at all.

### Why One Listener at the Root, Not One Per Element

React does not attach a real `addEventListener('click', ...)` to
every DOM node with an `onClick` prop. Instead, it attaches a small
number of listeners near the root of the app and relies on native
event bubbling — a click deep in the tree bubbles up to the root
listener, and React figures out which component's handler(s) along
that path should fire, in order, by walking its own fiber tree.

This is why: (1) thousands of `onClick` props don't mean thousands of
real DOM listeners (much cheaper), and (2) `event.stopPropagation()`
inside a React handler stops *React's* synthetic propagation, a
related but distinct mechanism from stopping the native DOM event.

### Why a Function Reference, Not a Call

``` jsx
onClick={handleClick()}   // ❌ calls handleClick immediately, during render
onClick={handleClick}     // ✅ passes the function itself, to be called later
```

`{handleClick()}` is a JavaScript expression evaluated the instant
this JSX line runs — which is *during render*, not when the user
clicks. Whatever `handleClick()` *returns* becomes the value of the
`onClick` prop (usually `undefined`), not a callable handler. React
never sees a function to attach; it sees the leftover return value.
This is a direct consequence of "JSX embeds JavaScript expressions,
evaluated eagerly" (see JSX notes) — nothing React-specific is
happening, it's plain JavaScript evaluation order catching people off
guard.

### Traced Example

``` text
User clicks <button onClick={handleClick}>
  → native click event fires on the real DOM node
  → bubbles to React's root listener
  → React walks fiber tree to find handlers along the path
  → constructs a SyntheticEvent, invokes handleClick(syntheticEvent)
  → handleClick calls setCount(c => c + 1)
  → update is queued on that fiber
  → after the handler function returns, React processes the batched update
  → re-render → reconciliation → commit
```

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
