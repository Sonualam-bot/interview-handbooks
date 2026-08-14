# 19_Render_Phase_vs_Commit_Phase_Notes

## Core Definition

React's update work can be understood as two major phases:

``` text
Render Phase
     ↓
Calculate what the UI should look like
     ↓
Commit Phase
     ↓
Apply the required changes
```

> **Render determines what the UI should look like. Commit applies the
> necessary changes to the host environment, such as the browser DOM.**

## 1. Render Phase

During rendering, React performs work such as:

-   Executing components
-   Reading current props and state
-   Creating new React Elements
-   Reconciling the new result with the previous tree
-   Determining what work needs to be committed

``` text
Component
    ↓
Execute
    ↓
New React Elements
    ↓
Reconciliation
    ↓
Determine required changes
```

The render phase is primarily about **calculation**.

## 2. Render Does Not Mean DOM Mutation

A component can render without the browser DOM changing.

``` jsx
function App() {
  const [count, setCount] = useState(0);

  return <div>Hello</div>;
}
```

If `count` changes but isn't used in the returned UI:

``` text
count: 0 → 1
```

React may execute the component again, create new React Elements, and
reconcile them, while requiring no relevant DOM mutation.

Therefore:

> **A React re-render does not necessarily mean a browser DOM update.**

## 3. Commit Phase

After React determines the required work, it enters the commit phase.

The commit phase applies the resulting changes to the host environment.

For a browser application, the host environment is the DOM.

Example:

``` text
Previous:

<h1>0</h1>

New:

<h1>1</h1>
```

Render determines the change:

``` text
0 → 1
```

Commit applies the corresponding DOM mutation.

## 4. Complete Counter Example

``` jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h1>{count}</h1>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

After clicking:

``` text
setCount(1)
    ↓
React schedules an update
    ↓
Render phase
    ↓
Counter executes again
    ↓
count = 1
    ↓
New React Elements
    ↓
Reconciliation
    ↓
Determine h1 content changed
    ↓
Commit phase
    ↓
DOM mutation
```

## 5. State Update Does Not Mean Immediate Rendering

Avoid:

``` text
setState()
    ↓
Immediately render
```

Prefer:

``` text
setState()
    ↓
React schedules an update
    ↓
React performs render work
```

This distinction becomes important for Fiber, scheduling, batching,
concurrent rendering, and priorities.

## 6. Render Can Be Repeated

Rendering is not necessarily guaranteed to run exactly once.

Conceptually:

``` text
started
   ↓
paused
   ↓
resumed
```

or:

``` text
started
   ↓
abandoned
   ↓
restarted
```

This becomes particularly important with Fiber and concurrent rendering.

## 7. Render Should Be Pure

Components should behave like pure functions during rendering.

``` text
props + state
      ↓
Component
      ↓
UI description
```

Avoid side effects directly inside render:

``` jsx
function Component() {
  fetch("/api/data"); // ❌
  return <div>Hello</div>;
}
```

Use appropriate mechanisms such as event handlers or effects for side
effects.

## 8. Render vs Reconciliation vs Diffing vs Commit

### Render

React performs rendering work and calculates the next UI.

### Reconciliation

React determines how the new result relates to the previous tree.

### Diffing

The comparison process used during reconciliation to determine relevant
changes.

### Commit

React applies the resulting changes to the host environment.

``` text
Render
  ↓
Reconciliation / Diffing
  ↓
Commit
  ↓
DOM / Host Mutations
```

## 9. Why Separate Render and Commit?

Separating calculation from application of changes gives React
flexibility.

Conceptually:

``` text
Render
   ↓
Calculate work
   ↓
Pause / resume / abandon if necessary
   ↓
Eventually commit
```

This becomes especially important for Fiber, scheduling, concurrent
rendering, and rendering priorities.

## 10. Connection to Fiber

Fiber allows React to represent rendering work as smaller units.

Conceptually:

``` text
Render Work
    ↓
Fiber A
    ↓
Fiber B
    ↓
Fiber C
    ↓
...
```

This allows React's scheduler to reason about rendering work and
prioritize it.

## Interview Answer

**Q: What is the difference between the render phase and commit phase?**

> The render phase is where React executes components, produces the next
> React Element tree, reconciles it with the previous tree, and
> determines what work needs to be performed. The commit phase is where
> React applies the resulting changes to the host environment, such as
> the browser DOM.

## Interview Nuggets

-   Render calculates; Commit applies.
-   Rendering a component does not necessarily mutate the DOM.
-   A state update schedules rendering work; it should not be thought of
    as an immediate synchronous DOM update.
-   Components should keep render logic pure.
-   Render work may be repeated.
-   Side effects should not be performed directly during render.
-   Reconciliation/diffing helps determine the work needed before
    commit.
-   Fiber makes rendering work interruptible and schedulable.

## Common Mistakes

❌ Every render causes a DOM update.

✅ A render can result in no DOM mutation.

❌ `setState` immediately renders the component.

✅ A state update schedules an update; React then performs render work.

❌ The render phase changes the DOM.

✅ The commit phase applies the required host mutations.

❌ Render is guaranteed to run exactly once.

✅ Rendering work can be repeated.

❌ Side effects belong inside component render.

✅ Keep rendering pure and use appropriate effect/event mechanisms for
side effects.

## Flashcards

**Q:** What happens during the render phase?

**A:** React executes components, creates the next React Elements,
reconciles the result, and determines required work.

**Q:** What happens during the commit phase?

**A:** React applies the resulting changes to the host environment, such
as DOM mutations in a browser.

**Q:** Does every render cause a DOM update?

**A:** No. Rendering can produce the same UI and therefore require no
DOM mutation.

**Q:** Why should render be pure?

**A:** React may perform rendering work more than once, so side effects
in render can execute unexpectedly or repeatedly.

**Q:** What does setState do conceptually?

**A:** It schedules an update; React then performs render work using the
updated state.

## 30-Second Revision

``` text
setState()
    ↓
Schedule Update
    ↓
┌─────────────────────────┐
│      RENDER PHASE       │
│                         │
│ Execute components      │
│ Create React Elements   │
│ Reconcile / Diff        │
│ Determine required work │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│      COMMIT PHASE       │
│                         │
│ Apply required host     │
│ mutations               │
└────────────┬────────────┘
             ↓
            DOM
```

Remember:

> **Render calculates. Commit applies.**

And:

> **A React re-render does not necessarily mean a DOM update.**
