# 20_Fiber_Architecture_Notes

## Core Definition

Fiber is React's internal architecture for representing and managing rendering work as units of work.

A Fiber is an internal data structure representing a component or element in React's rendering system and storing information React needs to manage identity, state, tree relationships, and rendering work.

> **Fiber does not mean multiple JavaScript threads. It gives React a unit-of-work model that makes rendering work more manageable, interruptible, and schedulable.**

## Why Fiber Was Needed

Large React trees can involve significant rendering work. Fiber lets React represent rendering as smaller units of work rather than treating the entire tree as one indivisible operation.

```text
Unit of Work
     ↓
Unit of Work
     ↓
Unit of Work
     ↓
...
```

## What Is a Fiber?

Conceptually:

```js
{
  type: Counter,
  key: null,

  stateNode: ...,

  child: ...,
  sibling: ...,
  return: ...,

  memoizedState: ...,
  memoizedProps: ...,

  alternate: ...
}
```

This is a simplified mental model, not an exact current implementation.

The important idea is that a Fiber stores information React needs to manage a component's identity, state, relationships, and work.

## Fiber Tree

For:

```jsx
<App>
  <Header />
  <Main />
  <Footer />
</App>
```

Conceptually:

```text
             App
              │
            child
              ↓
           Header
              │
          sibling
              ↓
            Main
              │
          sibling
              ↓
           Footer
```

Fibers are connected through relationships such as:

- `child`
- `sibling`
- `return`

## `child`, `sibling`, and `return`

For:

```jsx
<div>
  <Header />
  <Dashboard />
  <Footer />
</div>
```

Conceptually:

```text
       div
        |
      child
        ↓
     Header
        |
    sibling
        ↓
   Dashboard
        |
    sibling
        ↓
     Footer
```

And:

```text
Header.return     → div
Dashboard.return  → div
Footer.return     → div
```

## React Element vs Fiber

### React Element

A React Element is a lightweight description of what the UI should look like.

```js
{
  type: Counter,
  props: {}
}
```

### Fiber

A Fiber is an internal representation of a unit of work that React uses to manage rendering.

```text
React Element
      ↓
Description of UI

Fiber
      ↓
Internal unit of rendering work
```

> **React uses React Element information to construct and manage its internal Fiber representation.**

## Fiber and Component State

The conceptual relationship is:

```text
Component Identity
       ↓
Fiber
       ↓
Hooks / Internal State
       ↓
useState
```

Fiber participates in the internal representation through which React retains component state across renders.

## Current Tree and Work-in-Progress Tree

React can conceptually maintain:

```text
Current Tree
     ↕
Work-in-Progress Tree
```

### Current Tree

Represents the currently committed UI.

### Work-in-Progress Tree

Represents the next version React is currently working toward.

Example:

```text
Current:

Counter
count = 0
```

React receives:

```text
setCount(1)
```

and can work toward:

```text
Work-in-Progress:

Counter
count = 1
```

Once the work is successfully committed, the new version becomes the current tree.

## The `alternate` Relationship

Fiber nodes can have an `alternate` relationship:

```text
Current Fiber
     ↕
Work-in-Progress Fiber
```

The alternate represents the corresponding Fiber in the other tree.

## Why Have a Work-in-Progress Tree?

The currently committed UI can remain stable while React works toward the next version.

```text
CURRENT
count = 0
   │
   │ React works
   ↓
WORK-IN-PROGRESS
count = 1
```

This becomes important for:

- Interruptible rendering
- Concurrent rendering
- Scheduling
- Prioritization

## Fiber Makes Rendering Work Interruptible

Suppose React has:

```text
Fiber 1
Fiber 2
Fiber 3
Fiber 4
...
Fiber 100
```

Conceptually:

```text
Fiber 1
   ↓
Fiber 2
   ↓
Fiber 3
   ↓
Pause
   ↓
Resume later
   ↓
Fiber 4
```

This unit-of-work model is one of the fundamental capabilities enabled by Fiber.

## Fiber Does Not Mean Threads

Important:

```text
Fiber ≠ Thread
```

Fiber does not mean each component gets a separate JavaScript thread.

Instead:

```text
Fiber
  ↓
Units of work
  ↓
Work can be organized and scheduled
```

## Fiber and Scheduler

These have different responsibilities.

### Fiber

Provides the internal units of work and data structure.

> **What work exists?**

### Scheduler

Helps determine when work should be performed.

> **When should this work happen?**

Therefore:

```text
Fiber
  ↓
Units of work

Scheduler
  ↓
When work should run
```

Fiber itself should not be described as the Scheduler.

## Fiber and Concurrent Rendering

Fiber provides the foundation for more flexible rendering:

```text
Lower-priority work
        ↓
Rendering starts
        ↓
Higher-priority interaction arrives
        ↓
React can prioritize urgent work
        ↓
Lower-priority work can continue later
```

The exact priority model comes later.

## Fiber Is More Than a Performance Feature

Avoid:

> "Fiber simply makes React faster."

Better:

> **Fiber is React's architecture for representing rendering work as units that can be processed and scheduled more flexibly.**

Performance is a consequence; the architectural capability is the important part.

## Connecting the Previous Chapters

```text
Reconciliation
      ↓
Determine how new and old trees relate
      ↓
Diffing
      ↓
Determine relevant changes
      ↓
Identity
      ↓
Determine what can be preserved
      ↓
Render / Commit
      ↓
Calculate work and apply it
      ↓
Fiber
      ↓
Represent rendering work as manageable units
```

Broader pipeline:

```text
State Update
     ↓
Schedule Work
     ↓
Fiber Tree
     ↓
Render Work
     ↓
Reconciliation
     ↓
Diff / Determine Work
     ↓
Commit
     ↓
DOM
```

## Interview Answer

**Q: What is React Fiber?**

> **Fiber is React's internal architecture for representing and managing rendering work as units of work. A Fiber stores information about a component or element, its relationships in the tree, state, and work being performed. This architecture allows React to organize rendering work in a way that supports interruption, prioritization, scheduling, and concurrent rendering.**

## Interview Nuggets

- Fiber is an internal unit-of-work data structure.
- Fiber nodes form an internal tree.
- `child`, `sibling`, and `return` represent important tree relationships.
- React Elements describe the desired UI.
- Fibers represent internal rendering work.
- Fiber participates in preserving component identity and state.
- React can maintain Current and Work-in-Progress representations.
- `alternate` connects corresponding Fiber representations.
- Fiber does not create multiple JavaScript threads.
- Fiber provides units of work; the Scheduler helps determine when work should happen.
- Fiber is foundational to interruptible and concurrent rendering.

## Common Mistakes

❌ Fiber is the Virtual DOM.

✅ Fiber is an internal architecture/data structure used to manage rendering work.

❌ Fiber creates a separate thread for every component.

✅ Fiber organizes work within JavaScript's execution environment.

❌ Fiber itself is the Scheduler.

✅ Fiber provides units of work; scheduling is handled by React's scheduling mechanisms.

❌ Fiber simply means React renders faster.

✅ Fiber changes how React represents and manages rendering work, enabling more flexible scheduling and rendering.

## Flashcards

**Q:** What is a Fiber?

**A:** An internal React data structure representing a unit of rendering work.

**Q:** What are `child`, `sibling`, and `return`?

**A:** Internal relationships used to connect Fiber nodes into a tree.

**Q:** What is the difference between a React Element and a Fiber?

**A:** A React Element describes what the UI should look like; a Fiber is an internal representation React uses to manage rendering work.

**Q:** What is the Current Tree?

**A:** The currently committed representation of the UI.

**Q:** What is the Work-in-Progress Tree?

**A:** The representation of the next version React is currently working toward.

**Q:** Does Fiber create multiple JavaScript threads?

**A:** No.

**Q:** What does Fiber enable?

**A:** More flexible management of rendering work, including interruption, prioritization, scheduling, and concurrent rendering.

## 30-Second Revision

```text
React Element
     ↓
Description of UI

Fiber
     ↓
Internal unit of rendering work
     ↓
Fiber Tree
     ↓
Identity + State + Relationships + Work
     ↓
Current / Work-in-Progress
     ↓
Flexible Rendering
```

Remember:

> **Fiber does not make React multithreaded. It gives React a unit-of-work architecture that makes rendering work more manageable, interruptible, and schedulable.**

And:

> **Fiber represents the work; the Scheduler helps determine when that work should happen.**
