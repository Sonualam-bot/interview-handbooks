# 18_Keys\_&\_Identity_Notes

## Definition

Keys participate in determining component identity during
reconciliation.

Component state is associated with component identity, so changing
identity can cause state to reset.

## Deep Dive `[NEW]`

### Every Element Has an Implicit Key, Whether You Write One or Not

This is the detail that makes "same type + same position → same
identity" precise instead of hand-wavy: when you don't pass an
explicit `key`, React uses the element's **index within its parent's
children** as its implicit key. That's not a separate rule from the
explicit-key mechanism — it's the same mechanism, with React silently
filling in the identity value for you. This is exactly why reordering
an unkeyed list breaks state: the implicit key (the index) is
precisely what changes when you reorder.

### Why a Full Unmount/Remount, Not a Partial State Reset

When identity changes, React doesn't try to salvage anything from the
old instance — full unmount, full fresh mount. This looks aggressive,
but it's the only safe option: React has no way to know which pieces
of the old component's internal state (arbitrary `useState`/`useRef`/
`useReducer` values, possibly holding timers, subscriptions, DOM
node references) correspond to anything meaningful for a
*conceptually different* entity. There is no generally safe way to
"partially" carry state across an identity change, so React always
does the conservative thing: run cleanup, destroy everything, then
build fresh.

### Traced Example: Why `key={userId}` Resets Everything, Not Just the Prop

``` jsx
function Profile({ userId }) {
  const [tab, setTab] = useState('posts');   // unrelated to userId
  useEffect(() => {
    const sub = subscribe(userId);
    return () => sub.cancel();
  }, [userId]);
  ...
}

<Profile key={userId} userId={userId} />
```

When `userId` changes, the `key` changes too → React treats this as a
brand-new component identity → the *entire* fiber is discarded,
including `tab`, even though `tab` had nothing to do with `userId`.
That's the real trade-off of the "reset via key" pattern: it's a
blunt instrument — everything resets, not just the state you wanted
reset — which is exactly why it's reached for deliberately (to force
a clean slate) rather than as a targeted fix for one specific piece
of state.

## Core Mental Model

``` text
Component Type
      +
Key / Position
      ↓
Component Identity
      ↓
Component Instance / Internal State
```

If identity is preserved, existing state can be preserved.

If identity changes, the old component is unmounted and a new component
is mounted with fresh state.

## Same Type + Same Position

Without keys:

``` jsx
{showA ? <Counter /> : <Counter />}
```

Both branches produce `Counter` at the same position, so React can treat
them as the same identity and preserve state.

## Same Type + Different Key

``` jsx
{showA ? (
  <Counter key="A" />
) : (
  <Counter key="B" />
)}
```

Changing `A` to `B` changes identity.

``` text
Counter A
   ↓
Unmount
   ↓
State discarded

Counter B
   ↓
Mount
   ↓
Fresh state
```

## Keys and Lists

Stable keys allow React to match children even when their positions
change.

``` text
A → A
B → B
C → C
```

The corresponding component identities and state can therefore be
preserved.

## Index Keys

Index keys can cause identity problems when lists are reordered,
inserted into, or deleted from.

``` text
Before:
0 → A
1 → B
2 → C

After inserting X:
0 → X
1 → A
2 → B
3 → C
```

An existing component identity can become associated with a different
logical item, causing local state to appear to move.

## Keys Are Special

`key` is used by React for reconciliation and is not available as
`props.key`.

If the component needs the ID as data:

``` jsx
<Todo
  key={todo.id}
  id={todo.id}
/>
```

Use `id` as the prop.

## Keys Are Scoped to Siblings

Keys only need to be unique among the relevant siblings. They do not
need to be globally unique.

## Intentionally Resetting State

Keys can deliberately reset component state:

``` jsx
<Profile key={userId} />
```

When `userId` changes, the key changes, so React treats the new Profile
as a different identity.

``` text
Old Profile
   ↓
Unmount
   ↓
State discarded

New Profile
   ↓
Mount
   ↓
Fresh state
```

## Connection to useState

``` text
Component Identity
       ↓
Internal React Representation
       ↓
Fiber
       ↓
Hooks Associated With That Fiber
       ↓
useState State
```

Preserving identity allows React to find existing state. Changing
identity means the previous state is no longer used for the new
component.

## Interview Nuggets

-   Keys participate in component identity.
-   State is associated with component identity.
-   Changing a key can reset state.
-   Stable keys preserve identity across list reordering.
-   Keys are scoped to siblings.
-   `key` is a special React field.
-   Index keys are risky for dynamic lists.
-   Keys can intentionally reset component state.

## Interview Answer

**Q: Why does changing a key reset state?**

Changing the key changes the component's identity. React therefore
treats the new element as a different component, unmounts the previous
instance and its associated state, and mounts a new instance with fresh
state.

## Common Mistakes

-   State does not simply belong permanently to the component function;
    it is associated with component identity in the rendered tree.
-   Different source-code branches do not automatically mean different
    component instances.
-   Keys are not merely for removing console warnings; they provide
    stable identity during reconciliation.

## Flashcards

**Q:** What does a key provide?

**A:** Stable identity among siblings during reconciliation.

**Q:** What happens when a key changes?

**A:** The component receives a different identity, so the previous
state is discarded and a new instance is mounted.

**Q:** Why are index keys risky?

**A:** Reordering or inserting items can cause an existing component
identity to become associated with a different logical item.

**Q:** Can keys intentionally reset state?

**A:** Yes. `<Profile key={userId} />` can reset Profile state when the
user changes.

**Q:** Are keys globally unique?

**A:** No. They only need to be unique among the relevant siblings.

## 30-Second Revision

``` text
Stable Key
    ↓
Stable Identity
    ↓
Same Component Instance
    ↓
State Preserved
```

``` text
Changed Key
    ↓
Different Identity
    ↓
Old Instance Unmounts
    ↓
State Discarded
    ↓
New Instance Mounts
    ↓
Fresh State
```

Remember:

> **Keys are an identity mechanism, not merely a list-rendering
> feature.**
