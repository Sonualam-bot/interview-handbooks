# 13_Lifting_State_Up_Notes

## Definition

Lifting State Up is the process of moving state to the closest common
ancestor so multiple child components can share the same source of
truth.

------------------------------------------------------------------------

## Mental Model

``` text
User Interaction
      ↓
Child Event
      ↓
Parent State Updates
      ↓
Parent Re-renders
      ↓
Updated Props Flow Down
      ↓
Children Re-render
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

### Why Lift State?

-   Sibling components cannot directly access each other's state.
-   Shared data should have a single owner.

### Single Source of Truth

``` text
App
│
├── SearchBar
└── ProductList
```

`App` owns `query`.

Both children receive it via props.

### One-Way Data Flow

``` text
Parent
   ↓
Props
   ↓
Child
```

Sibling communication always goes through the common parent.

------------------------------------------------------------------------

## Execution Flow

1.  User types into `SearchBar`.
2.  Browser dispatches an input event.
3.  React invokes `handleChange`.
4.  `setQuery()` updates state in `App`.
5.  React stores the new state.
6.  `App` executes again.
7.  New props are passed to `SearchBar` and `ProductList`.
8.  A new React Element tree is created.
9.  Reconciliation compares old and new trees.
10. Commit updates the DOM.

------------------------------------------------------------------------

## Interview Nuggets

-   State should have one owner.
-   Lift state to the closest common ancestor.
-   Children receive shared state through props.
-   Siblings communicate through their parent.

------------------------------------------------------------------------

## Common Mistakes

❌ Duplicate the same state in multiple components.

✅ Keep one source of truth.

❌ Siblings share state directly.

✅ Shared state lives in the common parent.

------------------------------------------------------------------------

## Flashcards

**Q:** What is lifting state up?

**A:** Moving shared state to the closest common parent.

**Q:** Why can't siblings share state directly?

**A:** Each component instance owns its own local state.

**Q:** How does ProductList receive updated data?

**A:** Through updated props from the parent.

------------------------------------------------------------------------

## 30-Second Revision

-   One owner for shared state.
-   Lift state to the closest common ancestor.
-   Parent owns state.
-   Children receive props.
-   State update → Parent re-render → New props → Reconciliation →
    Commit.
