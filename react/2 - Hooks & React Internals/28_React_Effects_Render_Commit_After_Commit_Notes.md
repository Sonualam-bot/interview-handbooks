# 28 — Effects: Render vs Commit vs After Commit

## Interview Importance

**Priority: High.**

This chapter covers one of the most commonly misunderstood React topics:

- Side effects
- Render purity
- `useEffect`
- Dependency arrays
- Cleanup
- Effects vs event handlers
- Derived values vs effects
- `useLayoutEffect`
- Strict Mode and effect cleanup
- Render → Commit → Effect lifecycle

The key mental model:

```text
State Update
    ↓
Render
    ↓
Reconciliation
    ↓
Commit
    ↓
DOM mutations
    ↓
Effect
```

---

## 1. What Is a Side Effect?

A side effect is work that interacts with something outside the component's pure rendering calculation.

Examples:

```text
API request
Timer
Subscription
Logging
DOM interaction
Analytics
WebSocket
Event listener
Third-party library
```

Example:

```jsx
useEffect(() => {
  fetch("/api/users");
}, []);
```

The API request is a side effect.

---

## 2. Render Should Be Pure

React rendering should primarily answer:

> **What should the UI look like for this state and these props?**

Example:

```jsx
function Counter({ count }) {
  return <h1>{count}</h1>;
}
```

Why does purity matter?

React may render more than once. Rendering work can be:

```text
Started
 ↓
Paused
 ↓
Resumed
```

or:

```text
Started
 ↓
Abandoned
```

Therefore, render should not perform irreversible external side effects.

---

## 3. Why Side Effects Shouldn't Be in Render

Bad:

```jsx
function App() {
  sendAnalyticsEvent();

  return <Dashboard />;
}
```

React could render:

```text
Render #1
 ↓
Analytics event

Render #2
 ↓
Analytics event

Render #3
 ↓
Analytics event
```

even though the user only performed one meaningful action.

A render could also be abandoned before becoming visible UI.

---

## 4. `useEffect`

React provides:

```jsx
useEffect(...)
```

for synchronizing React with external systems.

Example:

```jsx
useEffect(() => {
  fetch("/api/users");
}, []);
```

Useful mental model:

```text
Render
 ↓
Commit
 ↓
Effect work
```

The exact timing has nuances, but the important interview distinction is:

> **Effects are not part of the pure render calculation.**

---

## 5. Render vs Effect

### Render

```jsx
return <h1>{count}</h1>;
```

Answers:

> What should the UI look like?

### Effect

```jsx
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]);
```

Answers:

> What external system needs to be synchronized with this rendered state?

---

## 6. Dependency Array

```jsx
useEffect(() => {
  console.log("effect");
}, [count]);
```

The dependency array describes the reactive values the effect depends on.

Conceptually:

```text
count unchanged
    ↓
Effect doesn't need to re-synchronize

count changed
    ↓
Effect runs again
```

---

## 7. Empty Dependency Array

```jsx
useEffect(() => {
  console.log("effect");
}, []);
```

For the common lifecycle mental model:

```text
Mount
 ↓
Effect
```

It does not mean "literally only once under every development behavior." Development Strict Mode can intentionally perform extra effect setup/cleanup cycles to expose bugs.

---

## 8. No Dependency Array

```jsx
useEffect(() => {
  console.log("effect");
});
```

Conceptually:

```text
Render
 ↓
Commit
 ↓
Effect

Next render
 ↓
Commit
 ↓
Effect
```

So it can run after every commit.

---

## 9. Cleanup

Effects can return a cleanup function:

```jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log("tick");
  }, 1000);

  return () => {
    clearInterval(id);
  };
}, []);
```

Mental model:

```text
Setup
 ↓
Effect active
 ↓
Cleanup
```

Cleanup reverses the setup performed by the effect.

---

## 10. Why Cleanup Matters

Without cleanup, resources can accumulate:

```text
Subscription 1
Subscription 2
Subscription 3
...
```

This can cause:

- Memory leaks
- Duplicate event handlers
- Duplicate subscriptions
- Unexpected behavior

---

## 11. Effect Lifecycle

Useful mental model:

```text
Effect setup
     ↓
Effect active
     ↓
Dependencies change / component unmounts
     ↓
Cleanup
     ↓
New setup if necessary
```

Example:

```jsx
useEffect(() => {
  const connection = connect(roomId);

  return () => {
    connection.disconnect();
  };
}, [roomId]);
```

If `roomId` changes:

```text
Cleanup A
   ↓
Setup B
```

---

## 12. Effects vs Derived Data

Don't put every calculation into an effect.

Avoid:

```jsx
useEffect(() => {
  setFullName(firstName + " " + lastName);
}, [firstName, lastName]);
```

Prefer:

```jsx
const fullName = firstName + " " + lastName;
```

if `fullName` is simply derived from existing state/props.

Mental model:

```text
State / props
 ↓
Calculation
 ↓
UI
```

No effect is required.

---

## 13. Event Logic vs Effects

If something happens directly because of a user action, an event handler is often the better place.

Prefer:

```jsx
function handleSubmit() {
  sendOrder();
}
```

rather than using an effect just to detect that a submission happened.

Think:

```text
User action
 ↓
Event handler
```

versus:

```text
Rendered state
 ↓
Synchronize external system
 ↓
Effect
```

---

## 14. Effect as Synchronization

A powerful mental model:

> **`useEffect` is primarily about synchronizing React with something outside React.**

Examples:

```text
React ↔ Browser API
React ↔ WebSocket
React ↔ Subscription
React ↔ Timer
React ↔ External library
React ↔ Network
```

This is more useful than thinking only:

> "`useEffect` runs after render."

---

## 15. Example: Event Listener

```jsx
useEffect(() => {
  function handleResize() {
    console.log(window.innerWidth);
  }

  window.addEventListener("resize", handleResize);

  return () => {
    window.removeEventListener("resize", handleResize);
  };
}, []);
```

Effect:

```text
Setup
 ↓
Add listener
```

Cleanup:

```text
Remove listener
```

---

## 16. Example: WebSocket

```jsx
useEffect(() => {
  const socket = new WebSocket(url);

  socket.onmessage = handleMessage;

  return () => {
    socket.close();
  };
}, [url]);
```

When `url` changes:

```text
Old socket
 ↓
Cleanup
 ↓
Close

New URL
 ↓
Setup
 ↓
New socket
```

---

## 17. Strict Mode and Effects

In development Strict Mode, React can intentionally perform an extra:

```text
Setup
 ↓
Cleanup
 ↓
Setup
```

cycle for effects.

The goal is to expose missing cleanup logic.

Correct pattern:

```jsx
useEffect(() => {
  window.addEventListener("resize", handler);

  return () => {
    window.removeEventListener("resize", handler);
  };
}, []);
```

---

## 18. Why Effects Connect to Concurrent Rendering

Recall:

```text
Render
 ↓
Can be interrupted
 ↓
Can be restarted
 ↓
Can be abandoned
```

Therefore:

```text
Render
```

must not contain irreversible external side effects.

Instead:

```text
Render
 ↓
Determine UI
 ↓
Commit
 ↓
Effect synchronization
```

This separation makes React's rendering model safer.

---

## 19. `useEffect` vs `useLayoutEffect`

### `useEffect`

Used for effects that don't need to block browser painting.

```jsx
useEffect(() => {
  // synchronize external system
});
```

### `useLayoutEffect`

Runs synchronously after DOM mutations but before the browser paints.

Useful when you need to:

- Measure layout
- Read DOM dimensions
- Make DOM adjustments before paint

Example:

```jsx
useLayoutEffect(() => {
  const rect = element.getBoundingClientRect();
}, []);
```

---

## 20. Don't Overuse `useLayoutEffect`

Because `useLayoutEffect` can block painting, prefer:

```text
useEffect
```

unless you specifically need synchronous DOM measurement/layout behavior before paint.

Mental model:

```text
useEffect
→ Usually preferred

useLayoutEffect
→ Layout-sensitive synchronous work
```

---

## 21. Effect Timing Mental Model

For interview purposes:

```text
Render
 ↓
DOM mutations during commit
 ↓
Browser can paint
 ↓
useEffect
```

Timing can have nuances depending on the interaction and React's scheduling.

The important distinction is:

```text
useEffect
→ Passive effect

useLayoutEffect
→ Layout-sensitive effect before paint
```

---

## 22. The Biggest `useEffect` Mistake

Anti-pattern:

```jsx
useEffect(() => {
  setFullName(firstName + " " + lastName);
}, [firstName, lastName]);
```

This can create:

```text
Render
 ↓
Effect
 ↓
setState
 ↓
Another render
```

when the value could simply be calculated during render:

```jsx
const fullName = firstName + " " + lastName;
```

---

## 23. Render → Commit → Effect

Key lifecycle:

```text
STATE UPDATE
     ↓
RENDER
     ↓
RECONCILIATION
     ↓
COMMIT
     ↓
DOM UPDATED
     ↓
EFFECTS
```

When an effect needs to be replaced:

```text
Previous effect
     ↓
Cleanup
     ↓
New effect setup
```

---

## 24. Interview: Why Shouldn't Side Effects Happen During Render?

Strong answer:

> **Because React rendering should be pure and React may render, restart, or abandon rendering work. If a side effect occurs during render, it could execute multiple times or execute for a render that never gets committed. Effects are used to synchronize with external systems outside the pure render calculation.**

---

## 25. Interview: What Is `useEffect` For?

Strong answer:

> **`useEffect` is used to synchronize a component with external systems such as subscriptions, timers, network requests, browser APIs, or third-party libraries. The effect runs outside the pure rendering calculation and can return a cleanup function to undo its setup.**

---

## 26. Interview: What Does the Dependency Array Do?

Example:

```jsx
useEffect(() => {
  // ...
}, [count]);
```

Answer:

> **It tells React which reactive values the effect depends on, so React can determine when the effect needs to be synchronized again.**

---

## 27. Interview: Why Do Effects Need Cleanup?

Strong answer:

> **Cleanup prevents resources created by an effect from remaining active after they are no longer needed. This is important for subscriptions, event listeners, timers, WebSockets, and similar resources.**

---

## 28. Common Mistakes

### ❌ `useEffect` is just for running code after render.

✅ Better mental model: it synchronizes React with external systems.

### ❌ Everything should go inside `useEffect`.

✅ Derived values and event-specific logic usually belong elsewhere.

### ❌ Effects are part of render.

✅ Effects happen separately from the pure render calculation.

### ❌ Side effects are safe during render because React only renders once.

✅ React may render multiple times, restart rendering, or abandon work.

### ❌ `useEffect([])` means literally "exactly once forever."

✅ It has no changing dependencies, but development Strict Mode can intentionally run extra setup/cleanup cycles.

### ❌ `useLayoutEffect` should always be preferred.

✅ Prefer `useEffect` unless synchronous DOM measurement/layout behavior requires it.

---

## 29. What You Actually Need to Know for Interviews

### Must Know

```text
Render
→ Calculate UI

Event Handler
→ Respond directly to user action

useEffect
→ Synchronize with external systems

Cleanup
→ Undo effect setup

Dependency array
→ Describes reactive values the effect depends on

useLayoutEffect
→ Synchronous DOM/layout-sensitive work
```

### Very Important Anti-Pattern

Avoid:

```text
state
 ↓
useEffect
 ↓
set derived state
 ↓
another render
```

when the value can simply be calculated during render.

### Don't Memorize

You do not need to memorize every internal implementation detail of effect scheduling.

Focus on:

```text
WHY effects exist
WHEN to use them
WHEN NOT to use them
HOW cleanup works
```

---

## 30. 30-Second Revision

```text
Render
 ↓
Calculate UI
 ↓
Reconciliation
 ↓
Commit
 ↓
DOM mutations
 ↓
Effect
```

Remember:

```text
Render
→ What should the UI look like?

Event handler
→ What should happen because the user did something?

Effect
→ What external system needs synchronization?

Cleanup
→ Undo the previous effect setup.
```

And:

```text
Render
≠
Effect
```

---

# Final Mental Model

> **React rendering should be pure: given the current props and state, determine what the UI should look like. Because rendering can be repeated, interrupted, restarted, or abandoned, irreversible side effects should not happen during render. `useEffect` provides a way to synchronize the committed React UI with external systems, while cleanup reverses the previous synchronization when necessary.**

The lifecycle:

```text
State Update
    ↓
Render
    ↓
Reconciliation
    ↓
Commit
    ↓
DOM
    ↓
Effect
```

And the interview rule:

> **If the logic calculates what the UI should look like → render.  
> If it responds directly to a user action → event handler.  
> If it synchronizes React with an external system → effect.**
