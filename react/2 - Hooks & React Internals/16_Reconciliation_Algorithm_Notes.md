# 16_Reconciliation_Algorithm_Notes

## Definition

Reconciliation is React's process of comparing the previous and newly
produced React Element trees to determine what changed and what can be
preserved.

The resulting changes are applied during the Commit phase.

## Mental Model

``` text
State Change
      ↓
Component Executes
      ↓
New React Element Tree
      ↓
Reconciliation
      ↓
Determine Identity / Changes
      ↓
Commit
      ↓
DOM Mutations
```

## Why Reconciliation Exists

React does not need to throw away the entire DOM whenever state changes.

``` jsx
<div>
  <h1>Hello</h1>
  <p>World</p>
</div>
```

becomes:

``` jsx
<div>
  <h1>Hello React</h1>
  <p>World</p>
</div>
```

The existing structure can be preserved and only the changed content
updated.

## Identity

Reconciliation asks:

> Does the new element represent the same thing as the previous element?

Important identity signals include:

-   Element type
-   Component type
-   Key for siblings
-   Position / sibling relationship

## Same Type

``` jsx
<div>Hello</div>
```

becomes:

``` jsx
<div>Hello React</div>
```

Both have `type = div`, so React can preserve the existing DOM structure
and update the changed content.

## Different DOM Type

``` jsx
<div>Hello</div>
```

becomes:

``` jsx
<section>Hello</section>
```

`div → section` represents a different identity, so React replaces the
relevant subtree.

## Same Component Type

``` jsx
<UserCard name="Sonu" />
```

becomes:

``` jsx
<UserCard name="Alex" />
```

The component type remains `UserCard`, so React can preserve the
component instance and state while providing new props.

## Different Component Type

``` jsx
<Profile />
```

becomes:

``` jsx
<AdminProfile />
```

Different component types mean different identities.

``` text
Profile
  ↓
Unmount
  ↓
State discarded

AdminProfile
  ↓
Mount
  ↓
Fresh state
```

Even if both return the same `<div>`, their component identities are
different.

## Reconciliation Is Recursive

React works through the tree.

``` text
div
├── Header
├── Main
└── Footer
```

If a new Footer is added:

``` text
Header → reuse
Main   → reuse
Footer → create
```

## Keys

Keys help React identify component instances among siblings.

``` jsx
[
  <Todo key="1" />,
  <Todo key="2" />,
  <Todo key="3" />
]
```

can be reordered while preserving identity:

``` jsx
[
  <Todo key="3" />,
  <Todo key="1" />,
  <Todo key="2" />
]
```

## Reconciliation vs Commit

**Reconciliation:** determines what changed.

**Commit:** applies the required DOM changes.

``` text
New Tree
   ↓
Reconciliation
   ↓
Determine Changes
   ↓
Commit
   ↓
DOM Mutations
```

## Important Distinction

### React Element

A description:

``` js
{
  type: Counter,
  props: {}
}
```

### Component Instance

Conceptually:

``` text
Counter instance
state = 10
```

### DOM Node

The actual browser object.

React Elements are recreated during rendering, while component
identity/state can be preserved during reconciliation.

## Interview Answer

Reconciliation is React's process of comparing the previous and newly
produced React Element trees to determine what has changed. React uses
element type, component type, keys, and positional/sibling relationships
to establish identity. When identity can be preserved, React reuses the
existing component or DOM structure; when identity changes, React
replaces the relevant subtree. The resulting changes are then applied
during the Commit phase.

## Interview Nuggets

-   Reconciliation determines what needs to change.
-   Commit applies the resulting DOM mutations.
-   Same component type can preserve state.
-   Different component types create different component identities.
-   Keys help preserve identity across sibling reordering.
-   React Elements are descriptions, not component instances or DOM
    nodes.

## Common Mistakes

-   Reconciliation does not directly update the DOM.
-   React does not reuse the same React Element object across renders.
-   Two components returning the same DOM are not necessarily the same
    component.

## Flashcards

**Q:** What is reconciliation?

**A:** Comparing previous and new React Element trees to determine what
changed.

**Q:** What happens when the component type changes?

**A:** React treats it as a different identity, unmounts the old
component, and mounts the new one.

**Q:** What happens to state when a component is replaced?

**A:** The old component's state is discarded and the new component
receives fresh state.

**Q:** Why are keys important?

**A:** They provide stable identity among siblings during
reconciliation.

**Q:** Does reconciliation update the DOM?

**A:** No. Reconciliation determines changes; Commit applies DOM
mutations.

## 30-Second Revision

``` text
State Update
    ↓
New React Element Tree
    ↓
Reconciliation
    ↓
Same identity? → Preserve
Different identity? → Replace
    ↓
Commit
    ↓
DOM Mutations
```

Remember:

> **React Elements are recreated; component identity and state can be
> preserved.**
