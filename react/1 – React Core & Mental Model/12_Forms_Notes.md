# 12_Forms_Notes

## Definition

Forms are the primary way users provide data to an application. In
React, form interactions follow the same rendering pipeline as every
other state update.

------------------------------------------------------------------------

## Deep Dive `[NEW]`

### Why Controlled Inputs Re-render On Every Keystroke (And Why That's Fine)

A controlled `<input value={query} onChange={...} />` means the DOM
input's displayed value is *always* whatever React last told it to
be — there's no independent "browser-owned" value the input can drift
to. That guarantee only holds if every keystroke round-trips through
React: keystroke → event → `setQuery` → re-render → new `value` prop
→ React writes it back into the DOM input. Skipping the re-render
would mean the DOM briefly holds a value React doesn't know about —
the *source of truth* would silently split (see Controlled vs
Uncontrolled).

This is why "typing is just another state update" (below) is the
load-bearing idea here: an input in React isn't a special case, it's
an ordinary controlled component that happens to render its own state
right back into its own `value` prop, closing the loop.

### The value/onChange Loop, Made Explicit

``` text
Render N:   <input value="Se" onChange={handleChange} />   (DOM shows "Se")
User types "t"
  → browser wants to show "Set", but the keystroke fires an onChange
    event instead of committing on its own
  → handleChange reads event.target.value → "Set" (the browser already
    computed what the value *would* be; React just reads it)
  → setQuery("Set")
  → re-render, component returns <input value="Set" .../>
  → React writes "Set" back into the DOM input's value property
DOM now shows "Set" — but only because React explicitly set it, not
because the browser's own edit was allowed to stick uncontested.
```

If `handleChange` called
`setQuery(event.target.value.toUpperCase())` instead, the DOM would
show `"SET"` even though the user typed lowercase — proof that the
DOM's displayed value is downstream of React's state, not the other
way around.

### Why preventDefault() on Submit

A `<form>`'s native default action is a full-page
navigation/reload — which would blow away all React state and the
whole SPA. `event.preventDefault()` stops that native browser
behavior so the submit event can be handled entirely in JS instead.

------------------------------------------------------------------------

## Mental Model

``` text
User Types
      ↓
Browser Event
      ↓
React SyntheticEvent
      ↓
Event Handler
      ↓
setState()
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

### Forms are Event Driven

``` jsx
<input onChange={handleChange} />
```

Typing generates a browser input event.

React receives the event and invokes the handler.

------------------------------------------------------------------------

### Event Object

``` jsx
function handleChange(event) {
    setQuery(event.target.value);
}
```

`event.target.value` contains the current value of the input.

------------------------------------------------------------------------

### React Element Before Typing

``` js
{
  type: "input",
  props: {
    value: "",
    onChange: handleChange
  }
}
```

After typing `"S"`:

``` js
{
  type: "input",
  props: {
    value: "S",
    onChange: handleChange
  }
}
```

Only the `value` prop changes.

------------------------------------------------------------------------

### Form Submission

``` jsx
<form onSubmit={handleSubmit}>
```

Use:

``` js
event.preventDefault();
```

to stop the browser's default page refresh.

------------------------------------------------------------------------

## Execution Flow

``` text
User Types
      ↓
Browser Dispatches Input Event
      ↓
React Creates SyntheticEvent
      ↓
handleChange(event)
      ↓
event.target.value
      ↓
setState()
      ↓
React Stores State
      ↓
Component Executes Again
      ↓
New React Element Tree
      ↓
Reconciliation
      ↓
Commit
```

------------------------------------------------------------------------

## Interview Nuggets

-   Forms are built on top of React events.
-   `event.target.value` contains the latest input value.
-   Typing is just another state update.
-   React receives browser events through its event system.

------------------------------------------------------------------------

## Common Mistakes

❌ Typing directly updates React.

✅ Typing dispatches an event; the event handler updates state.

❌ Forms update the DOM directly.

✅ State changes drive the UI update.

------------------------------------------------------------------------

## Flashcards

**Q:** What happens when a user types into an input?

**A:** Browser dispatches an input event → React invokes the handler →
state updates → component re-renders.

**Q:** What is `event.target.value`?

**A:** The current value of the input element.

**Q:** Why call `preventDefault()`?

**A:** To prevent the browser's default form submission behavior.

------------------------------------------------------------------------

## 30-Second Revision

-   Forms are driven by browser events.
-   React wraps them as SyntheticEvents.
-   Event handlers read `event.target.value`.
-   State updates trigger rendering.
-   Reconciliation computes minimal UI changes.
-   Commit updates the DOM.
