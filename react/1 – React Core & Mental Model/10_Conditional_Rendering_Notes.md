# 10_Conditional_Rendering_Notes

## Definition

Conditional rendering is the process of rendering different UI based on
application state. JavaScript evaluates the condition first; React
renders the result.

------------------------------------------------------------------------

## Deep Dive `[NEW]`

### There's No Such Thing As "React's Conditional Syntax"

This is the single most important thing to internalize about this
topic: React has no `if`/`else` construct of its own.
`{condition ? <A/> : <B/>}` works purely because JSX embeds
*JavaScript expressions* inside `{}` (see JSX notes: expressions, not
statements), and the ternary is a JavaScript expression that
evaluates to one value. `if` doesn't work inline because `if` is a
*statement* — it doesn't evaluate to a value that can sit inside `{}`.
That's exactly why `if` has to be pulled out above `return`, as a
statement, while the ternary, `&&`, and function calls all work
inline.

### The && Footgun, Explained From First Principles

`isAdmin && <AdminPanel />` relies on JavaScript's `&&` short-circuit:
if the left side is falsy, the expression evaluates to the left
side's *value*, not `false` specifically. This matters:

``` jsx
{count && <Badge count={count} />}
```

If `count` is `0`, `0 && <Badge />` evaluates to `0` — not `false`.
React renders `false`, `null`, `undefined`, and `true` as nothing, but
`0` is a valid, renderable value — so React prints a literal `0` on
the page. The fix, `{count > 0 && <Badge />}`, works because the left
side is now always a real boolean, never a number. This isn't a React
quirk to memorize as a "gotcha" — it's a direct consequence of
JavaScript's `&&` semantics combined with React's "falsy renders
nothing, but numbers are content" rule.

### Traced Example

``` text
loading = true
  ↓ JS evaluates: loading ? <Spinner/> : <Dashboard/>  →  <Spinner/> element
  ↓ React receives one element, builds that subtree

loading becomes false (via setLoading)
  ↓ component re-executes
  ↓ JS evaluates the same ternary → <Dashboard/> element
  ↓ different element type at this position → React discards the Spinner
    subtree, mounts Dashboard fresh (see Virtual DOM: different type = rebuild)
```

------------------------------------------------------------------------

## Mental Model

``` text
State
↓
Component Executes
↓
JavaScript Evaluates Condition
↓
React Element Tree
↓
Reconciliation
↓
Commit
↓
Real DOM
```

------------------------------------------------------------------------

## Core Concepts

### JavaScript Evaluates First

``` jsx
loading ? <Spinner /> : <Dashboard />
```

-   `loading = true` → `<Spinner />`
-   `loading = false` → `<Dashboard />`

React only receives the evaluated JSX.

### Conditional Rendering Creates Different Trees

``` text
loading = true
        ↓
Spinner Tree

loading = false
        ↓
Dashboard Tree
```

Reconciliation compares these trees.

### && Operator

``` jsx
isAdmin && <AdminPanel />
```

-   `true` → renders `<AdminPanel />`
-   `false` → React receives `false` and renders nothing.

### Returning null

``` jsx
if (!showBanner) return null;
```

`null` tells React to render nothing.

------------------------------------------------------------------------

## Execution Flow

``` text
setState()
↓
React Stores State
↓
Schedules Render
↓
Component Executes Again
↓
JavaScript Evaluates Condition
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

## Interview Nuggets

-   JavaScript evaluates conditions.
-   React renders the evaluated result.
-   `false`, `true`, `null`, and `undefined` are ignored during
    rendering.
-   State changes create new React Element trees.

------------------------------------------------------------------------

## Common Mistakes

❌ React evaluates conditions.

✅ JavaScript evaluates conditions before React creates React Elements.

❌ `false && <Component />` creates an empty component.

✅ It evaluates to `false`, which React ignores.

------------------------------------------------------------------------

## Flashcards

**Q:** Who evaluates JSX conditions?

**A:** JavaScript.

**Q:** Why does `false && <Component />` render nothing?

**A:** Because JavaScript returns `false`, which React ignores.

**Q:** What happens after a condition changes?

**A:** React creates a new React Element tree, reconciles it with the
previous tree, then commits DOM updates.

------------------------------------------------------------------------

## 30-Second Revision

-   Conditional rendering is powered by JavaScript.
-   React receives evaluated JSX.
-   Different conditions create different React Element trees.
-   Reconciliation compares trees.
-   Commit updates the DOM.
