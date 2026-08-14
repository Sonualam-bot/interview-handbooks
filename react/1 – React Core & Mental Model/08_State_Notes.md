# 08_State_Notes

## Definition

State is data owned by a component instance that can change over time
and trigger a re-render.

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
