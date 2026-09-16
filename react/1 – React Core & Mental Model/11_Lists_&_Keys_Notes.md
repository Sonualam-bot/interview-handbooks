# 11_Lists\_&\_Keys_Notes

## Definition

Lists are rendered by mapping data to React Elements. Keys provide a
stable identity for component instances during reconciliation.

------------------------------------------------------------------------

## Deep Dive `[NEW]`

### Why React Needs Keys At All (Not Just "Best Practice")

Every render, `.map()` produces a *brand-new array of brand-new
element objects* — even if the underlying data is identical, the
objects are not `===` to last render's objects (see React Elements:
elements are recreated, never mutated). Without any other
information, reconciling a list has only one way to match old
elements to new ones: **position in the array**. Position-based
matching is a *guess* — it assumes "whatever's at index 2 now
corresponds to whatever was at index 2 before," which is only true if
nothing was ever inserted, removed, or reordered.

`key` replaces that guess with an actual identity claim: "this
element, wherever it ends up, is the same conceptual thing as the
element that had this key last time." That's the entire purpose — not
a syntax requirement, but how React avoids a wrong guess about
identity.

### Why Wrong Guesses Are Dangerous, Concretely

Component instances (fiber nodes) hold state that's independent of
props (see State notes). If React wrongly matches "new index-2
element" to "old index-2 fiber" after an item was inserted at index
0, it reuses that fiber's *existing state* (a checkbox's checked
state, an input's typed text, a CSS transition's in-progress value)
for what is now conceptually a *different* list item. The DOM node's
identity doesn't change, so anything living in that node's
uncontrolled state (or on the fiber) sticks around and gets displayed
for the wrong data — the actual mechanism behind "wrong checkbox
selected" / "lost input focus," not just an observed symptom.

### Traced Example

``` text
Before: [A(key=1), B(key=2), C(key=3)]  → fibers: F1↔A, F2↔B, F3↔C
Insert X at front:
After:  [X(key=4), A(key=1), B(key=2), C(key=3)]

Keyed reconciliation:
  key=4 is new         → create new fiber F4
  key=1 matches F1     → reuse F1 (and its state) for A, now at index 1
  key=2 matches F2     → reuse F2 for B
  key=3 matches F3     → reuse F3 for C
Result: only one new fiber created; A/B/C keep their state exactly.

Index-based (no keys, or index-as-key):
  index 0 = X (was A)  → React thinks "A" turned into "X", reuses F1's state for X
  index 1 = A (was B)  → reuses F2's state for A
  ...
Result: every item after the insertion point silently inherits the wrong state.
```

------------------------------------------------------------------------

## Mental Model

``` text
Array Data
    ↓
map()
    ↓
Array of React Elements
    ↓
Virtual DOM
    ↓
Reconciliation
    ↓
Match by Key
    ↓
Commit
```

------------------------------------------------------------------------

## Core Concepts

### map() Returns React Elements

``` jsx
todos.map(todo => (
  <Todo key={todo.id} text={todo.text} />
))
```

returns an array of React Elements.

### What is a Key?

A key is a stable, unique identity among siblings used by React during
reconciliation.

Example:

``` js
{
  type: Todo,
  key: 1,
  props: {
    text: "Buy Milk"
  }
}
```

> `key` is metadata used by React. It is **not** available as
> `props.key`.

### Why Keys Matter

React recreates **React Elements** on every render.

Keys allow React to reuse the correct **component instances** across
renders.

### Why Not Index Keys?

Old:

``` text
0:A
1:B
2:C
```

Insert `X` at the beginning:

``` text
0:X
1:A
2:B
3:C
```

React now matches by index and may associate the previous component
state with the wrong item.

Result: - Wrong checkbox selected - Lost input focus - Incorrect form
values - Animation glitches

Use stable IDs instead.

------------------------------------------------------------------------

## Execution Flow

``` text
State Update
      ↓
Component Executes Again
      ↓
map()
      ↓
New React Element Tree
      ↓
Reconciliation
      ↓
Match by Key
      ↓
Reuse Component Instances
      ↓
Commit
```

------------------------------------------------------------------------

## Interview Nuggets

-   `map()` is JavaScript, not React.
-   Keys identify component instances, not React Elements.
-   React Elements are recreated every render.
-   Stable keys preserve state.

------------------------------------------------------------------------

## Common Mistakes

❌ Keys are for rendering lists.

✅ Keys preserve component identity during reconciliation.

❌ Index keys are always fine.

✅ Only use index keys for static lists that never change order.

❌ `key` is available as `props.key`.

✅ `key` is a special React field.

------------------------------------------------------------------------

## Flashcards

**Q:** Why do we need keys?

**A:** To give React a stable identity for component instances during
reconciliation.

**Q:** Why are index keys dangerous?

**A:** Reordering changes indexes, causing React to associate state with
the wrong component instance.

**Q:** Are React Elements reused?

**A:** No. React recreates React Elements every render and reuses
component instances based on keys.

------------------------------------------------------------------------

## 30-Second Revision

-   `map()` returns React Elements.
-   React recreates React Elements every render.
-   Keys preserve component identity.
-   Stable IDs \> array indexes.
-   Component identity preserves state.
