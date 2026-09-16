# 13_Lifting_State_Up_Notes

## Definition

Lifting State Up is the process of moving state to the closest common
ancestor so multiple child components can share the same source of
truth.

------------------------------------------------------------------------

## Deep Dive `[NEW]`

### Why Siblings Can't Just Talk To Each Other

Component state lives on a fiber, and a fiber only exposes its state
through the props it hands to its *own children* (one-way data flow,
see Props notes). Two sibling fibers have no reference to each
other at all — `SearchBar`'s fiber doesn't know `ProductList`'s fiber
exists. The only node with a reference to both is their common
parent, because it's the one that rendered both of them. So "shared
state must live in a common ancestor" isn't a guideline to follow —
it's a direct consequence of how data can physically flow through the
fiber tree: down through props, never sideways.

### Why Not Just Duplicate the State in Both Components?

If `SearchBar` and `ProductList` each kept their own local `query`
state, there would be two independent sources of truth that must be
kept manually in sync — precisely the "state/DOM drift" problem
React's whole render model exists to eliminate (see What is React),
recreated one level up, between two pieces of component state instead
of between state and DOM. Lifting state up applies the same
single-source-of-truth principle that motivates `UI = f(state)` in
the first place, at the granularity of a subtree instead of the whole
app.

### Traced Example

``` text
App (owns query, setQuery)
├── SearchBar(query, onQueryChange)     — receives query via props, calls
│                                          onQueryChange(newValue) on input
└── ProductList(query)                  — receives the SAME query via props,
                                           filters by it

User types in SearchBar's input
  → SearchBar calls onQueryChange("phone")   (a prop function, defined in
                                               App, passed down)
  → that prop function is literally App's setQuery
  → App's state updates → App re-renders
  → App passes query="phone" to BOTH children again
  → SearchBar shows "phone" in the input (controlled), ProductList re-filters
```

`onQueryChange` is not a special mechanism — it's an ordinary prop (a
callback function) passed down, the *only* tool a component has for
triggering a change in an ancestor's state: call a function the
ancestor gave you.

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
