# 17_Diffing_Algorithm_Notes

## Definition

React's diffing algorithm is the comparison process used during
reconciliation to efficiently determine how the previous and new React
Element trees differ.

React uses heuristics instead of a general-purpose tree comparison.

## Deep Dive `[NEW]`

### The Same Heuristic As Reconciliation, One Level Deeper: Why Not Just Ask "Are These Equal?"

A generic deep-equality check between the old and new tree would need
to recursively walk every node and compare every prop — and still
wouldn't tell React *which* nodes to reuse when structure shifts
(insertion, deletion, reordering). Equality checking answers "are
these the same," a binary question; reconciliation needs to answer
"which parts overlap, and how do the rest map," an alignment problem.
That's why React never does a `JSON.stringify`-style comparison
anywhere — it's the wrong kind of question. Structural/identity
heuristics (type + key + position) are the cheap proxy React uses to
answer the alignment question without full equality checking.

### Why a Single Forward Pass, Not a Two-Pointer or LCS Diff

A more thorough list-diffing approach (like the longest-common-
subsequence algorithm `git diff` uses on text) would correctly find
the true minimal set of moves for an arbitrarily reordered list.
React deliberately doesn't do this — its list reconciliation is a
single forward pass matching by key, plus a limited move-detection
heuristic. Fast (O(n)), but degenerate reorderings (reversing a very
long list) can be diagnosed as "many moves" rather than the true
minimal set. In practice, key-stable lists rarely reorder in ways
that justify the extra algorithmic cost of a proper LCS diff on every
render.

### Traced Example: Where the Heuristic Gets a "Wrong" Answer On Purpose

``` text
Old: <div><Header/><Sidebar/></div>
New: <div><Sidebar/><Header/></div>   (no keys)
```

A human sees this as a swap — reuse both, only reorder. Without keys,
React's per-index comparison sees: index 0 was `Header`, now
`Sidebar` → different type → unmount Header, mount Sidebar. Index 1
was `Sidebar`, now `Header` → different type → unmount Sidebar, mount
Header. Both components remount with fresh state, even though nothing
conceptually needed destroying — the algorithm intentionally never
searches beyond "what's at this same index" without a key telling it
to.

## Core Assumptions

### 1. Different element types produce different trees

``` jsx
<div>Hello</div>
```

becomes:

``` jsx
<span>Hello</span>
```

Different types mean different identities, so React replaces the
relevant subtree.

### 2. Keys provide stable identity for children

Keys allow React to match corresponding children even when their
positions change.

## Same Type

``` jsx
<div>Hello</div>
```

becomes:

``` jsx
<div>Hello React</div>
```

Both have `type = div`.

React can preserve the existing DOM structure and update the changed
content.

## Different Component Types

``` jsx
<UserCard />
```

becomes:

``` jsx
<AdminCard />
```

Different component types mean different identities.

``` text
UserCard
   ↓
Unmount

AdminCard
   ↓
Mount
```

Even if both components return the same DOM structure, their component
identities are different.

## Same Component Type

``` jsx
<UserCard name="Sonu" />
```

becomes:

``` jsx
<UserCard name="Alex" />
```

The component type remains `UserCard`.

React can preserve the component identity and state while providing new
props.

## Children Comparison

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

React compares:

``` text
div → div
h1  → h1
p   → p
```

and then discovers:

``` text
"Hello" → "Hello React"
```

Only the changed part needs to be updated.

## Lists and Keys

Previous:

``` jsx
[
  <Todo key="1" text="A" />,
  <Todo key="2" text="B" />,
  <Todo key="3" text="C" />
]
```

New:

``` jsx
[
  <Todo key="3" text="C" />,
  <Todo key="1" text="A" />,
  <Todo key="4" text="D" />
]
```

Conceptually React can match:

``` text
key=1 → A → A       preserve
key=2 → B → —       delete
key=3 → C → C       preserve
key=4 → — → D       create
```

The order change does not require the matching component instances to
lose their identity because the keys remain stable.

## Important Distinction

React **does create new React Elements during a new render**.

What can be preserved is the **component identity / Fiber and its
state** when reconciliation finds a match.

``` text
New Render
    ↓
New React Elements
    ↓
Diffing / Reconciliation
    ↓
Matching identity
    ↓
Existing component identity preserved
    ↓
Existing state preserved
```

## Reconciliation vs Diffing

### Reconciliation

The broader process of determining how the new tree relates to the
previous tree.

### Diffing

The comparison process used during reconciliation to determine what
changed.

``` text
Reconciliation
      ↓
Diffing / Comparison
      ↓
Determine Changes
      ↓
Commit
      ↓
DOM Mutations
```

## Interview Answer

React uses heuristics to efficiently compare the previous and new React
Element trees instead of performing a general-purpose tree comparison.
It considers element type to determine whether a subtree can be
preserved, while keys provide stable identity for children in lists. For
matching elements, React continues comparing props and children; when
identity changes, the relevant subtree is replaced. The resulting
changes are then applied during the Commit phase.

## Interview Nuggets

-   Diffing happens during reconciliation.
-   Different element types generally mean different identities.
-   Same component type can preserve component identity and state.
-   Keys provide stable identity among siblings.
-   React Elements are recreated on every render.
-   Component identity and state can be preserved after matching.
-   Reconciliation determines changes; Commit applies DOM mutations.

## Common Mistakes

❌ React performs a generic deep equality check between the two trees.

✅ React uses structural and identity heuristics.

❌ React reuses the same React Element object.

✅ New React Elements are created on each render.

❌ Moving a keyed item means its component instance must be recreated.

✅ Stable keys allow React to preserve the corresponding identity.

## Flashcards

**Q:** What does the diffing algorithm compare?

**A:** The previous and new React Element trees to determine changes.

**Q:** What happens when element types differ?

**A:** React treats them as different identities and replaces the
relevant subtree.

**Q:** Why are keys important?

**A:** They provide stable identity for children and allow React to
match items across renders.

**Q:** Does React recreate React Elements?

**A:** Yes. New React Elements are created during each render.

**Q:** What can React preserve?

**A:** Matching component identity / Fiber and its associated state.

## 30-Second Revision

``` text
Old Tree + New Tree
        ↓
Diffing during Reconciliation
        ↓
Compare Type / Key / Structure
        ↓
Same Identity → Preserve
Different Identity → Replace
        ↓
Commit
        ↓
DOM Mutations
```

Remember:

> **React recreates Elements; stable identity lets reconciliation
> preserve component instances and state.**
