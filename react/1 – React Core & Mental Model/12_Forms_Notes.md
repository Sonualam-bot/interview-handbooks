# 12_Forms_Notes

## Definition

Forms are the primary way users provide data to an application. In
React, form interactions follow the same rendering pipeline as every
other state update.

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
