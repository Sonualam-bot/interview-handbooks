# 11_Lists\_&\_Keys_Notes

## Definition

Lists are rendered by mapping data to React Elements. Keys provide a
stable identity for component instances during reconciliation.

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
