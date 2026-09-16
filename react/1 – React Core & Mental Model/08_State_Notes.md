# 08_State_Notes

## Definition

State is data owned by a component instance that can change over time
and trigger a re-render.

------------------------------------------------------------------------

## Deep Dive `[NEW]`

### Why State Can't Just Be a Local Variable

A local variable (`let count = 0`) inside a function is
re-initialized every time the function runs — and a component
function *does* run again on every render (that's how `UI = f(state)`
works). If React stored nothing outside the function call, no value
could survive from one render to the next. So state, definitionally,
has to live somewhere that outlives a single execution of the
component function: on the fiber node, not in the function's local
scope.

### Where useState Actually Stores Things

Each fiber keeps a linked list (`memoizedState`) of hook state, one
entry per hook call, in call order. `useState(0)` doesn't "create"
state each render — on mount it initializes one linked-list entry; on
every subsequent render it walks to the *next* entry in that same
list and returns whatever is currently stored there. This is the real
mechanism behind "don't call hooks conditionally": hooks are matched
to their stored state purely by call order/position, not by name — an
`if` around a `useState` call would misalign every hook after it with
the wrong stored slot.

``` text
Render 1: useState(0) → creates list node #1, value 0 → returns [0, setCount]
Render 2: useState(0) → reads list node #1 (unchanged), value 5 → returns [5, setCount]
```

The `0` passed to `useState(0)` is only ever used on the *first*
render — every subsequent render, React ignores the argument entirely
and returns whatever is already in the slot.

### Why setCount "Schedules" Instead of Updating Immediately

If `setCount` mutated the DOM synchronously inside the event handler,
calling it three times in one handler would cause three separate
re-renders and three DOM writes for what is conceptually one user
action. Instead, `setCount` marks the fiber as needing an update and
*schedules* work; React batches every state update within the same
event handler (or transition) into a single re-render, computed once
the handler finishes. This is why reading `count` immediately after
calling `setCount` in the same function still shows the old value —
the update hasn't been applied yet, only queued.

### Traced Example

``` jsx
function Counter() {
  const [count, setCount] = useState(0);
  const handleClick = () => {
    setCount(count + 1);
    console.log(count); // still logs the OLD value
  };
  return <button onClick={handleClick}>{count}</button>;
}
```

`handleClick` is a closure created during *this* render, so `count`
inside it is frozen at whatever value this render saw. `setCount`
doesn't change that closure's `count` — it schedules a new render,
which creates a brand-new `handleClick` closure with the updated
`count` baked in.

------------------------------------------------------------------------

## Mental Model

``` text
State Update
      ↓
React Stores New State
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

### Why State?

-   Makes UI dynamic.
-   Persists across renders.
-   Managed by React, not by local variables.

### State vs Variables

``` js
let count = 0;
count++;
```

Changing a normal variable does **not** notify React.

``` jsx
const [count, setCount] = useState(0);
```

Calling `setCount()` stores the new state and schedules another render.

### Where Does State Live?

Conceptually, React stores state outside the component so it survives
multiple renders.

### State Belongs to Component Instances

``` jsx
<Counter />
<Counter />
```

Each instance owns independent state.

### Initial Render vs Re-render

Initial Mount: - Read initial state - Build React Element tree - Commit

Update: - Store new state - Execute component again - Build new tree -
Reconciliation - Commit

------------------------------------------------------------------------

## Interview Nuggets

-   State belongs to the component instance.
-   React owns the state storage.
-   Updating state schedules a render.
-   State is the source of truth for interactive UI.

------------------------------------------------------------------------

## Common Mistakes

❌ State lives inside the component function.

✅ React stores state and provides it during each render.

❌ Updating a local variable re-renders the UI.

✅ Only state updates notify React.

❌ State directly updates the DOM.

✅ State starts React's rendering pipeline.

------------------------------------------------------------------------

## Flashcards

**Q:** What is state?

**A:** Data owned by a component instance that can change and trigger
re-renders.

**Q:** Where is state stored?

**A:** Conceptually by React, outside the component function.

**Q:** Why doesn't `count++` update the UI?

**A:** React isn't notified of local variable changes.

------------------------------------------------------------------------

## 30-Second Revision

-   State is owned by the component instance.
-   React stores state.
-   `setState` schedules a render.
-   Components execute again.
-   React builds a new React Element tree.
-   Reconciliation finds changes.
-   Commit updates the DOM.
